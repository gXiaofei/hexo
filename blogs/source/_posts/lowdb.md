---
title: lowdb
date: 2018-08-08 10:12:22
tags:
---

# LowDB

​	源地址 ： https://www.npmjs.com/package/lowdb

> 用于Node，Electron和浏览器的小型JSON数据库。由Lodash提供技术支持

```javascript
db.get('posts')
  .push({ id: 1, title: 'lowdb is awesome'})
  .write()
```

## 用法

```javascript
// 安装
npm install lowdb
```

```javascript
const low = require('lowdb');
const FileSync = require('lowdb/adapters/FileSync');

const adapter = new FileSync('db.json'); // 存储到db.json文件中
const db = low(adapter);	

// 设置一些默认值
db.defaults({posts: [], user: {}}).write();

// 添加一个post
db.get('posts').push({id: 1, title: 'lowdb is awesome'}).write();

// 使用Lodash简写语法设置用户
db.set('user.name', 'typicode').write();
```

数据保存到`db.json`

```javascript
{
  "posts": [
    { "id": 1, "title": "lowdb is awesome"}
  ],
  "user": {
    "name": "typicode"
  }
}
```

你可以使用任何[lodash](https://lodash.com/docs)函数[`_.get`](https://lodash.com/docs#get)和[`_.find`](https://lodash.com/docs#find)简写语法

```javascript
// 如果您只是从db读取，请使用.value（）而不是.write（）
db.get('posts').find({id: 1}).value();
```

Lowdb非常适合CLI，小型服务器，Electron应用程序和npm软件包。

它支持**Node**，**浏览器**并使用**lodash API**，因此学习起来非常简单。实际上，如果你知道Lodash，你已经知道如何使用lowdb

- 用法示例
  - [CLI](https://github.com/typicode/lowdb/tree/master/examples#cli)
  - [浏览器](https://github.com/typicode/lowdb/tree/master/examples#browser)
  - [服务器](https://github.com/typicode/lowdb/tree/master/examples#server)
  - [In-memory](https://github.com/typicode/lowdb/tree/master/examples#in-memory)
- [JSFiddle实例](https://jsfiddle.net/typicode/4kd7xxbu/)

**重要的** lowdb不支持群集，并且可能存在非常大的JSON文件（~200MB）的问题。

在[unpkg](https://unpkg.com/)上也可以使用UMD构建进行测试和快速原型设计：

```javascript
<script src="https://unpkg.com/lodash@4/lodash.min.js"></script>
<script src="https://unpkg.com/lowdb@0.17/dist/low.min.js"></script>
<script src="https://unpkg.com/lowdb@0.17/dist/LocalStorage.min.js"></script>
<script>
  var adapter = new LocalStorage('db')
  var db = low(adapter)
</script> 
```

## API

1. low(adapter)

   返回具有下面描述的其他属性和函数的lodash [链](https://lodash.com/docs/4.17.4#chain) 

2. db.[...].write()           db.[...].value()

   `write()`是一个语法糖，用于调用`value()`和`db.write()`一行。 

   另一方面，`value()`只是[_.protoype.value（）](https://lodash.com/docs/4.17.4#prototype-value)并且应该用于执行不更改数据库状态的链。 

   ```javascript
   db.set('user.name', 'typicode').write()
   
   // 相当于
   db.set('user.name', 'typicode').value()
   db.write()
   ```

3. db._

   数据库lodash实例。使用它来添加自己的实用程序函数或第三方mixins，如[underscore-contrib](https://github.com/documentcloud/underscore-contrib)或[lodash-id](https://github.com/typicode/lodash-id)。 

   ```javascript
   db._.mixin({
       second: function(array){
           return array[1]
       }
   })
   
   db.get('posts').second().value()
   ```

4. db.getState()

   返回数据库状态

   ```javascript
   db.getState() // {posts:[...]}
   ```

5. db.setState(nweState)

   替换数据库状态  db.json 内容没变

   ```javascript
   const newState = {}
   db.setState(newState)
   ```

6. db.write()

   使用持久化数据库`adapter.write`（取决于adapter，可能会返回一个promise）。 

   ```javascript
   // 使用 lowdb/adapter/FileSync  同步
   db.write()
   console.log('state has been saved'); // 状态已保存
   
   // 使用lowdb/adapter/FileAsync
   db.write().then(()=>console.log('state has been saved'))
   ```

7. db.read()

   使用`storage.read`选项读取源代码（取决于adapter，可能会返回一个promise） 

   ```javascript
   // 使用 lowdb/FileSync
   db.read()
   console.log('State has been updated') // 状态已更新
   
   // 使用 lowdb/FileAsync
   db.read().then(()=>console.log('State has been updated'))
   ```

## apapter API

请注意，这仅适用于与Lowdb捆绑在一起的适配器。第三方适配器可能有不同的选项。 

为了方便，`FileSync`，`FileAsync`和`LocalBrowser`接受以下选项： 

- **defaultValue**  如果文件不存在，则该值将被用于设定初始状态（default：`{}`） 
- **serialize/deserialize**  在写入之前和读取之后使用的函数（default：`JSON.stringify`和`JSON.parse`）    序列化/反序列化 

```javascript
const adapter = new FileSync('array.yaml', {
  defaultValue: [],
  serialize: (array) => toYamlString(array),
  deserialize: (string) => fromYamlString(string)
})
```

## 指南

使用Lowdb，您可以访问整个lodash API，因此有很多方法来查询和操作数据

请注意，数据是通过引用返回的，这意味着对返回对象的修改可能会更改数据库。为了避免这种行为，您需要使用`.cloneDeep()`. 

此外，方法的执行是懒惰的，也就是说，延迟执行直到调用`value()`或.`write()`



检查posts是否存在

```javascript
db.has('posts').value()
```



设置posts

```javascript
db.set('posts', []).write()
```



排序前五个 posts (文章)

```javascript
db.get('posts')
	.filter({published: true})
	.sortBy('views')
	.task(5)
	.value()
```



获取文章标题

```javascript
db.get('posts)
	.map('title')
	.value()
```



获取文章数量

```javascript
db.get('posts')
	.size()
	.value()
```



使用路径获取第一篇文章的标题。

```javascript
db.get('posts[0].title')
	.value()
```



更新一个post

```javascript
db.get('posts')
	.find({title: 'low!'})
	.assign({title: 'hi!'})
	.write()
```



删除 posts

```javascript
db.get('posts')
	.remove({title: 'low!'})
	.write()
```



移除属性

```javascript
db.unset('user.name')
	.write()
```



深克隆 posts

```javascript
db.get('posts')
	.cloneDeep()
	.value()
```



## 如何使用基于id的资源

能够使用id获取数据非常有用，尤其是在服务器中。要向lowdb添加基于id的资源支持，您有2个选项 

1. [shortid](https://github.com/dylang/shortid)更简约，并返回一个可在创建资源时使用的唯一ID。 

   ```javascript
   const shortid = require('shortid')
    
   const postId = db
     .get('posts')
     .push({ id: shortid.generate(), title: 'low!' })
     .write()
     .id
    
   const post = db
     .get('posts')
     .find({ id: postId })
     .value()
   ```

2. [lodash-id](https://github.com/typicode/lodash-id)提供了一组帮助程序，用于创建和操作基于id的资源。 

   ```javascript
   const lodashId = require('lodash-id')
   const db = low('db.json')
    
   db._.mixin(lodashId)
    
   const post = db
     .get('posts')
     .insert({ title: 'low!' })
     .write()
    
   const post = db
     .get('posts')
     .getById(post.id)
     .value(
   ```

## 如何创建自定义的适配器

`low()` 接受自定义适配器，因此您可以使用任何格式将数据虚拟地保存到任何存储。 

```javascript
class MyStorage  { 
  constructor （） { 
    //  ...
  }
 
  read （） { 
    //  应返回数据（对象或数组）或Promise
  }
 
  write(data){ 
    //  不应该返回任何内容或Promise
  }
}
 
const adapter = new MyStorage（ args ）    
const db = low （）
```

See [src/adapters](https://github.com/typicode/lowdb/blob/HEAD/src/adapters) for examples.   查看例子

## 如何加密数据

`FileSync`，`FileAsync`和`LocalStorage`接受自定义`serialize`和`deserialize`功能。您可以使用它们来添加加密逻辑。 

```javascript
const adapter = new FileSync('db.json', {
  serialize: (data) => encrypt(JSON.stringify(data))  // 加密
  deserialize: (data) => JSON.parse(decrypt(data))  // 解密
})
```

