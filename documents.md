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

这种方法包含首先从Mongo中索引文档，然后发出一个更新命令（通过调用`save`触发）。