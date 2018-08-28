## 子文档（Subdocuments）
子文档是嵌入在其他文档中的文档，在Mongoose中这意味着可以在其他schema中嵌套schema。Mongoose中有两个不同的子文档概念：子文档数组和单个嵌套子文档。

```js
var childSchema = new Schema({ name: 'string' });

var parentSchema = new Schema({
  // Array of subdocuments
  children: [childSchema],
  // Single nested subdocuments. Caveat: single nested subdocs only work
  // in mongoose >= 4.2.0
  child: childSchema
});
```

子文档与普通文档类似。嵌套schema具有中间件，自定义验证逻辑，虚拟属性，和任意其他的可被使用的顶级schema特点。主要的区别是子文档不是被单独保存的，子文档在它的顶级父文档保存时保存。

```js
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';

// `parent.children[0].save()` is a no-op, it triggers middleware but
// does **not** actually save the subdocument. You need to save the parent
// doc.
parent.save(callback);
```

子文档类似顶层文档有保存和验证的中间件。在父文档调用`save()`会触发所有它子文档的`save()`中间件，`validate`中间件也一样。

```js
childSchema.pre('save', function (next) {
  if ('invalid' == this.name) {
    return next(new Error('#sadpanda'));
  }
  next();
});

var parent = new Parent({ children: [{ name: 'invalid' }] });
parent.save(function (err) {
  console.log(err.message) // #sadpanda
});
```

子文档的`pre('save')`和`pre('validate')`中间件在顶层文档的`pre(save)`之前和`pre(validate)`之后执行。这是因为在`save()`之前的验证实际是一些内建的中间件。

```js
// Below code will print out 1-4 in order
var childSchema = new mongoose.Schema({ name: 'string' });

childSchema.pre('validate', function(next) {
  console.log('2');
  next();
});

childSchema.pre('save', function(next) {
  console.log('3');
  next();
});

var parentSchema = new mongoose.Schema({
  child: childSchema,
    });

parentSchema.pre('validate', function(next) {
  console.log('1');
  next();
});

parentSchema.pre('save', function(next) {
  console.log('4');
  next();
});
```

### 查找一个subdocument
每个子文档默认有一个`_id`字段。Mongoose文档数组有一个特殊的id方法，用于搜索文档数组，以便于查找具有给定`_id`的文档。

```js
var doc = parent.children.id(_id);
```

### 添加子文档到数组中
Mongoose数组方法类似push，unshift，addToSet等显示的转化参数成自己的属性类型：

```js
var Parent = mongoose.model('Parent');
var parent = new Parent;

// create a comment
parent.children.push({ name: 'Liesl' });
var subdoc = parent.children[0];
console.log(subdoc) // { _id: '501d86090d371bab2c0341c5', name: 'Liesl' }
subdoc.isNew; // true

parent.save(function (err) {
  if (err) return handleError(err)
  console.log('Success!');
});
```

子文档可以被Mongoose数组创建而不用添加进数组。

```js
var newdoc = parent.children.create({ name: 'Aaron' });
```

### 移除子文档
每个子文档有移除方法。对于数组子文档，它等价于在子文档中调用`.pull()`。对于单个嵌套子文档，`remove()`是等价于设置子文档为`null`。

```js
/ Equivalent to `parent.children.pull(_id)`
parent.children.id(_id).remove();
// Equivalent to `parent.child = null`
parent.child.remove();
parent.save(function (err) {
  if (err) return handleError(err);
  console.log('the subdocs were removed');
});
```

### 数组代替声明语法
如果使用对象数组创建一个schema，mongoose会自动转化对象成schema：

```js
var parentSchema = new Schema({
  children: [{ name: 'string' }]
});
// Equivalent
var parentSchema = new Schema({
  children: [new Schema({ name: 'string' })]
});
```

### 接下来
现在已经讲完子文档，让我们看一看查询。