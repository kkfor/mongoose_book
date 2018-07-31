## 模式（Schemas）
如果你目前没有接触过，请花一些时间阅读[快速上手](quickstart.md)了解mongoose是如何工作的。如果你是从4.x迁移到5.x，请花一些时间阅读迁移指南。
### 定义schema
Mongoose中一切开始于Schema。每个schema映射着MongoDB的集合，在集合中定义了文档的结构。

```js
  var mongoose = require('mongoose');
  var Schema = mongoose.Schema;

  var blogSchema = new Schema({
    title:  String,
    author: String,
    body:   String,
    comments: [{ body: String, date: Date }],
    date: { type: Date, default: Date.now },
    hidden: Boolean,
    meta: {
      votes: Number,
      favs:  Number
    }
  });
```

如果你想要添加额外的键，用Schema的add方法。  

在`blogSchema`中的每个键都定义了文档中的一个属性。每个键都将转换成关联的模式类型。例如，我们定义的`title`将被转换成字符串模式类型，date属性将被转换成`Date`模式类型。键也可以被分配包含深层的键/类型声明的嵌套对象，类似上面的meta属性。

允许的模式类型：
- String
- Number
- Date
- Buffer
- Boolean
- Mixed
- ObjectId
- Array
- Decimal128
- Map

更多关于[SchemaTypes](schematypes.md)

Schemas不仅定义文档的结构和属性的转换，而且还定义文档的实例方法，静态模型方法，复合索引和叫做中间件的文档生命周期钩子。

### 创建一个模型
用schema来定义，我们需要将`blogSchema`转换成可以工作的model，所以，我们通过`mogoose.model(modelName, schema)`来转换：

```js
  var Blog = mongoose.model('Blog', blogSchema);
  // ready to go!
```

### 实例方法
Models实例是文档。文档有很多它们内部的实例方法。我们也可以声明自定义的实例方法。

```js
  // 声明一个schema
  var animalSchema = new Schema({ name: String, type: String });

  // 在animalSchema的methods对象上分配一个函数
  animalSchema.methods.findSimilarTypes = function(cb) {
    return this.model('Animal').find({ type: this.type }, cb);
  };
```

现在所有的`animal`实例都有了一个可用的`findSimilarTypes`方法。
```js
  var Animal = mongoose.model('Animal', animalSchema);
  var dog = new Animal({ type: 'dog' });

  dog.findSimilarTypes(function(err, dogs) {
    console.log(dogs); // woof
  });
```

- 覆写一个默认的mongoose文档方法会导致不可预测的结果。点击这里查看详情。
- 上面的例子用`Schema.methods`对象直接保存实例方法。你还可以使用这里描述的Schema.method()。
- 不要用ES6的箭头(=>)函数声明方法。箭头函数明确阻止绑定`this`，所以方法不能访问文档，以上例子不能运行。

### 静态方法