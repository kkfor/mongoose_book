## 文档（Documents）
Mongoose文档表示储存在MongoDB数据库中文档的一对一映射。每个文档是一个Model的实例。

### 检索
从MongooDB中检索文档有很多种方法。在本节中不讨论这个问题。查看查询章节关于更多详细

### 更新
有许多方法可以更新文档。首先使用传统的方法findById：

```js
Tank.findById(id, function (err, tank) {
  if (err) return handleError(err);

  tank.size = 'large';
  tank.save(function (err, updatedTank) {
    if (err) return handleError(err);
    res.send(updatedTank);
  });
});
```

你也可以用`.set()`来修改文档。在下面，`tank.size = 'large';`修改为`tank.set({size: 'large'})`。

```js
Tank.findById(id, function (err, tank) {
  if (err) return handleError(err);

  tank.set({ size: 'large' });
  tank.save(function (err, updatedTank) {
    if (err) return handleError(err);
    res.send(updatedTank);
  });
});
```

这种方法包含首先从Mongo中索引文档，然后发出一个更新命令（通过调用`save`触发）。如果你不需要文档返回应用只想要直接更新数据库中的一个属性，Model#update可以满足我们：

```js
Tank.update({ _id: id }, { $set: { size: 'large' }}, callback);
```

如果我们需要文档返回应用，有一个更好的选择：

```js
Tank.findByIdAndUpdate(id, { $set: { size: 'large' }}, { new: true }, function (err, tank) {
  if (err) return handleError(err);
  res.send(tank);
});
```

`findAndUpdate/Remove`静态方法最多修改一个文档，在数据库中调用一次并返回它。findAndModify有几种变体，阅读API文档查看更多详情。  
注意`findAndUpdate/Remove`在修改数据库前不执行任何钩子或验证。你可以用`runValidators`选项访问有限的文档验证子集。然而你需要挂钩和完整的文档验证，首先查询文档和`save()`。

### 验证
文档在保存前被验证。阅读api文档或者验证章节查看更多。

### 覆写
你可以用`.set()`覆盖整个文档。如果你想要改变中间件中保存的文档是很方便的。

```js
Tank.findById(id, function (err, tank) {
  if (err) return handleError(err);
  // Now `otherTank` is a copy of `tank`
  otherTank.set(tank);
});
```

### 接下来
现在我们已经介绍完了文档，让我们看一看子文档。
