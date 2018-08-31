## 查询（Queries）
Mongoose模型对于增删改查操作提供了几个静态辅助函数。每个函数都返回一个mongoose查询对象。

- `Model.deleteMany()`
- `Model.deleteOne()`
- `Model.find()`
- `Model.findById()`
- `Model.findByIdAndDelete()`
- `Model.findByIdAndRemove()`
- `Model.findByIdAndUpdate()`
- `Model.findOne()`
- `Model.findOneAndDelete()`
- `Model.findOneAndRemove()`
- `Model.findOneAndUpdate()`
- `Model.replaceOne()`
- `Model.updateMany()`
- `Model.updateOne()`

mongoose查询可以用两种方式中的一种执行。如果传递回调函数，操作将会立即执行，结果将传递给回调函数。  
查询有一个`.then()`函数，因此可以作用promise使用。  
当使用回调函数进行查询时，将查询指定为JSON文档。JSON文档的语法和MongoDB shell的语法是相同的。

```js
var Person = mongoose.model('Person', yourSchema);

// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  // Prints "Space Ghost is a talk show host".
  console.log('%s %s is a %s.', person.name.first, person.name.last,
    person.occupation);
});
```
