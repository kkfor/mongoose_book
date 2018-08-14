## 模型（Models）
模型是根据Schema定义构造的奇妙构造函数。一个模型的实例是一个文档。Model是负责从底层MongoDB数据库。
- 编译你的第一个model
- 构建文档
- 查询
- 删除
- 更新
- 改变流

### 编译你的第一个model

```js
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```

第一个参数是模型的集合的单数名。Mongoose会自动寻找模型名字的复数版本。因此，上面的例子，模型Tank用于在数据库中tanks的集合。`.model()`函数复制`schema`。在调用`.model()`之前请确保在`schema`中已添加完全。

### 构造文档
一个model的实例是一个文档。创建和保存文档是简单的。

```js
var Tank = mongoose.model('Tank', yourSchema);

var small = new Tank({ size: 'small' });
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
});

// or

Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
});

// 或者用于插入大量文档
Tank.insertMany([{ size: 'small' }], function(err) {

});
```

注意，在模型连接打开之前，没有tanks被创建或移除。每个模型都有一个关联的连接。当你`mongoose.model()`时，你的model将使用默认的连接。

```js
mongoose.connect('localhost', 'gettingstarted');
```

如果你想创建自定义的连接，用连接中的`model()`函数来代替默认连接。

```js
var connection = mongoose.createConnection('mongodb://localhost:27017/test');
var Tank = connection.model('Tank', yourSchema);
```

### 查询
用Mongoose查找文档很简单，它支持MongoDB中丰富的查询语法。可以使用find，findById，findOne或其他的静态方法来检索文档。

```js
Tank.find({ size: 'small' }).where('createdDate').gt(oneYearAgo).exec(callback);
```

查看查询章节，关于使用查询api的更多详细的信息。

### 删除
模型有静态方法`deleteOne()`和`deleteMany()`方法用于删除所有过滤器匹配到的所有文档。

```js
Tank.deleteOne({ size: 'large' }, function (err) {
  if (err) return handleError(err);
  // 最多删除一个文档
});
```

### 更新
每个模型都有自己的更新方法来修改数据库中的档，而不是返回到你的应用程序。关于更多信息，查看api文档。

```js
Tank.updateOne({ size: 'large' }, { name: 'T-90' }, function(err, res) {
  // 最多更新一个文档，`res.modifiedCount`包含数量
  // MongoDB更新文档
});
```

如果你想要在数据库中更新一个文档并返回到你的应用，用findOneAndUpdate代替。

### 更改流
在MongoDB 3.6.0和Mongoose 5.0.0中的新特性  
更改流为你提供了一种方法，可以通过MongoDB数据库监听所有的插入和更新。注意除非你连接到MongoDB的副本集，否则更改流无法工作。

```js
async function run() {
  // 创建一个新的mongoose模型
  const personSchema = new mongoose.Schema({
    name: String
  });
  const Person = mongoose.model('Person', personSchema, 'Person');

  // 创建一个更改流。在数据库中有改变发生时将会触发'change'事件
  Person.watch().
    on('change', data => console.log(new Date(), data));

  // 插入一个文档，将会触发以上的更改流
  console.log(new Date(), 'Inserting doc');
  await Person.create({ name: 'Axl Rose' });
}
```

上面的异步函数输入结果如下所示。

```js
2018-05-11T15:05:35.467Z 'Inserting doc'
2018-05-11T15:05:35.487Z 'Inserted doc'
2018-05-11T15:05:35.491Z { _id: { _data: ... },
  operationType: 'insert',
  fullDocument: { _id: 5af5b13fe526027666c6bf83, name: 'Axl Rose', __v: 0 },
  ns: { db: 'test', coll: 'Person' },
  documentKey: { _id: 5af5b13fe526027666c6bf83 } }
```

你可以阅读在这篇博客文章中阅读更多关于mongoose更改流。

### 更多
API文档有更多额外的可用方法类似count，mapReduce，aggregate和更多。

### 接下来
现在我们已经学完了`Model`，让我们看一下文档。