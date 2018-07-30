# 起步

首先确定你安装了`MongoDB`和`Node.js`。  
接下来用`npm`在命令行中安装`Mongoose`:
```
$ npm install mongoose
```
我们喜欢小猫想要记录下在MongoDB中遇到的每只猫。首先我们需要在本地运行的MongoDB实例中，利用mongoose来连接`test`数据库。  

```js
// getting-started.js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');
```

在本地有一个对`test`数据库挂起的连接。当连接成功或失败我们需要得到提示：

```js
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  // 连接成功
});
```

一旦连接成功，回调函数就会运行。为了简单起见，假设后面的代码都是在回调中运行的。  
在Mongoose中，所有东西都是由`Schema`来派生的。让我们声明一个猫来作为参考来了解Schema。

```
var kittySchema = new mongoose.Schema({
  name: String
});
```