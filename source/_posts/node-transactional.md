---
title: Nodejs Service, Dao, transaction设计
date: 2017-02-16 14:36:59
tags: 
- nodejs
- transaction
- mysql
- continuation-local-storage
---
Nodes是单线程运行的，所有io操作是在另一个线程执行，处理完后通过回调函数通知nodejs线程，所以在dao层会出现回调，尤其是当一些业务逻辑或数据库操作需要顺序执行的时候，会产生层层回调，如何更好地复用代码，管理*transaction*，是一个问题。

#### router
```javascript
router.get('/add', async function(req, res, next) {
  try {
    var data = await userService.addUser();
    res.send(data)
  } catch (error) {
    next(error);
  }
});
```

router的回调是一个async方法，try catch 后，有error就调用`next(error);`让下个处理异常的*middleware*处理，没什么特别的

#### dao
```javascript
const sqls = {
    insert:'INSERT INTO user(id, name, age) VALUES(0,?,?)'
}

const dao = {
    add: function (name, age, connection) {
		return new Promise(function(resolve, reject) {
          connection.query(sqls.insert, [name,age], (err, data) => (err ? reject(err) : resolve(data)));
        })
	}
}

module.exports = dao;
```

dao所有方法都返回一个promise，connection需要调用方传进来（*暂时没想到解决办法*），因为同一个transaction需要用同一个connection。

#### service
```javascript
const userDao = require('../dao/userDao')
const dbHelper = require('../dao/dbHelper')

const service = {}

async function addUser(connection){
    var user1 = await userDao.add('pw',18,connection);
    var user2 = await userDao.add('tangjianfeng',99,connection);
    return {data:'success'}
}

service.addUser = dbHelper.transactionProxy(addUser,service)

module.exports = service;
```

想达到的效果是transaction统一加在每个service方法上，方法里任意地方抛错就*rollback*，有点java里spring的@transactional annotation的意思。首先定义addUser方法本身,他除了要接受router传进来的参数外，还要接受一个connection参数(会通过proxy的方式传进来)，将同一个connection传到不同的dao方法中，来控制transaction。

`service.addUser = dbHelper.transactionProxy(addUser,service)`

这句主要是给addUser方法加一层代理(实际router里调用的是proxy方法)。

#### dbHelper
```javascript
var mysql = require('mysql');
var $conf = require('../conf/db');

var pool  = mysql.createPool($conf.mysql);

const helper = {
    startTransaction(){
        return new Promise(function(resolve, reject) {
            pool.getConnection(function(err, connection) {
                if(err){ throw err;}
                connection.beginTransaction(function(err) {
                    if (err) { 
                        throw err; 
                    }else{
                        resolve(connection)
                    }
                })
            })
        })
    },
    transactionProxy(fn,context){
        var context = context || this;
        var manage = this;
        return async function(){
            //get connect and start transaction
            try {
                var connection = await manage.startTransaction();
                var arr = Array.from(arguments)
                arr.push(connection)
                var result = await fn.apply(context,arr)

                //commit
                connection.commit(function(err) {
                    if (err) {
                        connection.rollback(function() {
                            throw err;
                        });
                    }
                })

                return result;
            } catch(error){
                connection.rollback();
                throw error
            } finally {
                connection.release()
            }
        }
    }
}

module.exports = helper;
```

dbHelper是关键***transactionProxy***方法可以给fn外面包了一层，这一层主要作用是：

1. 从connectionPool中拿到拿到一个connection
2. connection开启transaction
3. 将这个connection作为额外参数传给fn方法，并调用fn方法
4. 拿到result后commit
5. try catch住，有异常就rollback
6. 最后finally中release connection

#### 好处

- 使用promise async await的写法，没有嵌套的回调，代码可读性强，易于维护
- transaction加在service层，这样dao层的方法可以复用(不用担心transaction)
- 写法简单，重复的 `beginTransaction` `commit` `rollback` 以及dao层的异常处理都可以统一放到`transactionProxy`中去处理。

#### 不足

- connection需要显示地传递，有依赖，解决思路是如何不需要传递connection,就能使得dao的方法使用同一个connection。

------

### decorator version

#### service

```javascript
const userDao = require('../../dao/userDao/userDao')
const transactional = require('../../dao/dbHelper').transactional

class LoginService {
  @transactional
  async getUserInfo(userId,session,connection){
    var user = await userDao.queryById(userId,connection);
    return user
  }
}

module.exports = LoginService
```

#### dbHelper

```javascript
helper.transactional = function (target, name, descriptor) {
  let originMethod = descriptor.value;
  let newMethod = async function () {
    //get connect and start transaction
    try {
      var connection = await helper.startTransaction();
      var arr = Array.from(arguments)
      arr.push(connection)
      var result = await originMethod.apply(target, arr)

      //commit
      connection.commit(function (err) {
        if (err) {
          connection.rollback(function () {
            throw err;
          });
        }
      })

      return result;
    } catch (error) {
      connection.rollback();
      throw error
    } finally {
      connection.release()
    }
  }
  descriptor.value = newMethod;
  return descriptor;
}
```

原理一样只是把之前的***transactionProxy***改成了decorator方式实现，类似java annotation，**service**里的代码更简洁了。

------

### continuation-local-storage version

[**continuation-local-storage**](https://github.com/othiym23/node-continuation-local-storage)的作用相当于java里的ThreadLocal,不同的是它是基于链式回调，而不是真正的thread(nodejs是单线程的)。使用**continuation-local-storage**,我们可以在整个链式回调方法的生命周期里，get、set该回调链独有的变量值，于是就可以做到不显示的传递connection，而在整个transaction里使用同一个connection了。

#### service

```javascript
const UserDao = require('../../dao/userDao/userDao')
const transactional = require('../../dao/dbHelper').transactional

var userDao = new UserDao()

class LoginService {
  @transactional
  async getUserInfo(userId,session){
    var user = await userDao.queryById(userId);
    return user
  }

  @transactional
  async addUsers(userName,session){
    var user = await userDao.add(userName,11);
    var user = await userDao.add(userName,12);
    return user
  }
}

module.exports = LoginService
```

service里只是把之前所有传递的connection去掉了。

#### dao

```javascript
const connection = require('../../dao/dbHelper').connection

const sqls = {
  insert: 'INSERT INTO user(id, name, age) VALUES(0,?,?)',
  queryById: 'select * from user where id=?',
  queryByName: 'select * from user where name=?',
  queryAll: 'select * from user'
}

class UserDao {
  @connection
  get connection (){}

  queryById (id) {
    var id = +id; // 为了拼凑正确的sql语句，这里要转下整数
    var self = this;
    return new Promise(function (resolve, reject) {
      self.connection.query(sqls.queryById, id, (err, data) => (err ? reject(err) : resolve(data)));
    })
  }

  add (name, age) {
    var self = this;
    return new Promise(function (resolve, reject) {
      if(name == 'daxianyu' && age == 12){
        return reject(new Error('daxianyu forbidden'))
      }
      self.connection.query(sqls.insert, [name, age], (err, data) => (err ? reject(err) : 				resolve(data)));
    })
  }
}

module.exports = UserDao;
```

dao改写成了class，因为需要加*@connection* decorator，@connection改写了connection属性的**get**方法,拿到当前回调链中定义的connection。

#### dbHelper

```javascript
var mysql = require('mysql');
var $conf = require('../conf/db');
var asyncLocal = require('continuation-local-storage');

var pool = mysql.createPool($conf.mysql);
var connectionLocal = asyncLocal.createNamespace('connectionLocal');
var connectionName = 'connection';

const helper = {
  startTransaction() {
    return new Promise(function (resolve, reject) {
      pool.getConnection(function (err, connection) {
        if (err) { throw err; }
        connection.beginTransaction(function (err) {
          if (err) {
            throw err;
          } else {
            resolve(connection)
          }
        })
      })
    })
  }
}

helper.transactional = function (target, name, descriptor) {
  let originMethod = descriptor.value;
  let newMethod = async function () {
    //get connect and start transaction
    try {
      let context = connectionLocal.createContext();
      connectionLocal.enter(context);
      var connection = await helper.startTransaction();
      connectionLocal.set(connectionName, connection);
      var arr = Array.from(arguments)
      arr.push(connection)
      var result = await originMethod.apply(target, arr)

      //commit
      connection.commit(function (err) {
        if (err) {
          connection.rollback(function () {
            throw err;
          });
        }
      })

      return result;
    } catch (error) {
      connection.rollback();
      throw error
    } finally {
      connection.release()
    }
  }
  descriptor.value = newMethod;
  return descriptor;
}

helper.connection = function (target, name, descriptor){
  descriptor.get = function() {
    return connectionLocal.get(connectionName);
  }
  return descriptor;
}

module.exports = helper;
```

*@transaction* decorator改写了一下，在开启transaction前加入了下面几行代码：

```javascript
let context = connectionLocal.createContext();
connectionLocal.enter(context);
...
connectionLocal.set(connectionName, connection);
```

开启一个新的context，把当前transaction连接保存到当前的**connectionLocal**中，之后的回调链中都能拿到同一个connection。
*@connection* decorator改写了**get**方法,返回当前**connectionLocal**中保存的connection。
这样**service**、**dao**的写法就十分简单，通过*@transactional*、*@connection*，两个decorator就可以控制事务。

#### 问题
重新测试发现*node-continuation-local-storage*不支持async function,我提了个[issue](https://github.com/othiym23/node-continuation-local-storage/issues/98),原来V8引擎的async实现方式有些特殊，V8 5.7，也就是Node 8会得到支持,本来Generator函数可以完美使用local storage,但是不支持decorator,所以暂时就这样吧。