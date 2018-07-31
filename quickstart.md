## 起步

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
在Mongoose中，所有东西都是由`Schema`来派生的。让我们声明一只猫来作为参考来了解Schema。

```js
var kittySchema = new mongoose.Schema({
  name: String
});
```
到目前一切很好，我们得到了一个schema，它拥有一个String类型的name属性。接下来将schema编译成model。
```js
var Kitten = mongoose.model('Kitten', kittySchema);
```
一个模型是一个类，他拥有我们构建的文档。在本例中，每个文档都是由schema声明带有属性和行为的小猫。让我们创建一个小猫文件代表刚才在外面遇到的小男孩。

```js
var silence = new Kitten({ name: 'Silence' });
console.log(silence.name); // 'Silence'
```

猫可以叫，让我们在文档中添加一个“speak”方法：

```js
// 注意: 必须在用mogoose.model()编译前在schema中添加方法
kittySchema.methods.speak = function () {
  var greeting = this.name
    ? "Meow name is " + this.name
    : "I don't have a name";
  console.log(greeting);
}

var Kitten = mongoose.model('Kitten', kittySchema);
```

在schema中添加方法属性编译为模型的原型暴露到每个文档实例：

```js
var fluffy = new Kitten({ name: 'fluffy' });
fluffy.speak(); // "Meow name is fluffy"
```

我们已经有了会叫的猫！但是没有在MongoDB中保存任何东西。每个文档都可以通过调用save方法保存到数据库中。回调的第一个参数是发生的错误。

```js
var fluffy = new Kitten({ name: 'fluffy' });
fluffy.speak(); // "Meow name is fluffy"
```

随着时间的流逝，我们想要展示见过的所有小猫。我们可以通过猫的模型访问所有猫文档。

```js
Kitten.find(function (err, kittens) {
  if (err) return console.error(err);
  console.log(kittens);
})
```

我们只是将数据库中所有的猫输出到了控制台。如果想要通过名字筛选所有猫，MongoDB支持丰富的查询语句。

```js
Kitten.find({ name: /^fluff/ }, callback);
```

它执行了对所有文档以“Fluff”开头的名字属性的搜索，通过回调返回一个猫的数组结果。

### 恭喜
这是起步的最后。我们在MongoDB中用mongoose创建了一个schema,添加了一个自定义的文档方法，然后保存和查询。前往指南，或api文档。