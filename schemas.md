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

如果你之后想要添加额外的键，用Schema#add方法。  

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
- 不要用ES6的箭头（=>）函数声明方法。箭头函数明确阻止绑定`this`，所以方法不能访问文档，以上例子不能运行。

### 静态方法
在`Model`上添加静态方法也很简单，继续在`animalSchema`上扩展：

```js
// 在animalSchema的“statics”对象上分配一个函数
animalSchema.statics.findByName = function(name, cb) {
  return this.find({ name: new RegExp(name, 'i') }, cb);
};

var Animal = mongoose.model('Animal', animalSchema);
Animal.findByName('fido', function(err, animals) {
  console.log(animals);
});
```

不要用ES6的箭头函数（=>）声明静态方法。箭头函数会明确阻止绑定`this`，因为`this`的值以上例子不能运行。

### 查询助手
你可以添加查询助手函数，对于mongoose查询它像是实例方法。查询助手方法可以扩展mongoose链查询api。

```js
animalSchema.query.byName = function(name) {
  return this.where({ name: new RegExp(name, 'i') });
};

var Animal = mongoose.model('Animal', animalSchema);

Animal.find().byName('fido').exec(function(err, animals) {
  console.log(animals);
});

Animal.findOne().byName('fido').exec(function(err, animal) {
  console.log(animal);
});
```

### 索引
MongoDB支持二级索引。在mongoose中，利用`Schema`在路径级别或`schema`级别定义索引。当创建复杂索引时，在schema级别定义所以是必要的。

```js
var animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true } // field level
});

animalSchema.index({ name: 1, type: -1 }); // schema level
```

当应用启动后，Mongoose在schema中为每个声明的索引自动调用`createIndex`。Mongoose将会按顺序调用每个索引的`createIndex`，当所有的`createIndex`调用成功或发生错误是，在模型上触发一个“index”事件，为了更好的运行，禁止在生产环境中运用此行为，创建索引会对性能造成明显的影响。通过将schema的`autoIndex`选项设置为`false`来禁用此行为,或者在连接的全局设置`autoIndex`为`false`。

```js
mongoose.connect('mongodb://user:pass@localhost:port/database', { autoIndex: false });
// or
mongoose.createConnection('mongodb://user:pass@localhost:port/database', { autoIndex: false });
// or
animalSchema.set('autoIndex', false);
// or
new Schema({..}, { autoIndex: false });
```

当索引创建或发生错误，Mongoose将会在模型上触发一个`index`事件。

```js
// 将会发生错误，因为mongodb默认有一个_id索引
// is not sparse
animalSchema.index({ _id: 1 }, { sparse: true });
var Animal = mongoose.model('Animal', animalSchema);

Animal.on('index', function(error) {
  // "_id index cannot be sparse"
  console.log(error.message);
});
```

参见模型#ensureIndexes方法。

### 虚拟属性
虚拟属性是可以获取和设置的文档属性但不会持久化到MongoDB中。getter对格式化和合并字段很有用，同时setter对将单个值分解为多个值储存很有用。

```js
// 定义一个schema
var personSchema = new Schema({
  name: {
    first: String,
    last: String
  }
});

// 编译model
var Person = mongoose.model('Person', personSchema);

// 创建一个文档
var axl = new Person({
  name: { first: 'Axl', last: 'Rose' }
});
```

如果你想打印出全名，你可以这样做：

```js
console.log(axl.name.first + ' ' + axl.name.last); // Axl Rose
```

但是每次连接姓和名时会很麻烦。如果你想在名字上做一些额外的处理，类似移除一些符号？一个虚拟属性getter可以做到。  
定义一个不会持久化到MongoDB的`fullName`属性。

```js
personSchema.virtual('fullName').get(function () {
  return this.name.first + ' ' + this.name.last;
});
```

现在，mongoose可以在你每次访问fullName属性时调用getter函数：

```js
console.log(axl.fullName); // Axl Rose
```
如果你用`toJson()`或者`toObject()`mongoose默认将不会包含虚拟属性。这包括调用Mongoose文档上的`JSON.stringify()`，因为`JSON.stringify()`调用了`toJSON`。将`{virtuals: true}`传递给`toObject()`或者`toJSON()`。  
你也可以在虚拟属性中添加一个自定义的setter，它将允许你用`fullName`虚拟属性同时设置姓和名。

```js
personSchema.virtual('fullName').
  get(function() { return this.name.first + ' ' + this.name.last; }).
  set(function(v) {
    this.name.first = v.substr(0, v.indexOf(' '));
    this.name.last = v.substr(v.indexOf(' ') + 1);
  });

axl.fullName = 'William Rose'; // Now `axl.name.first` is "William"
```

在其他验证前也可以用虚拟属性setter。所以上面的例子即使需要`first`和`last`字段，也可以运行。  
在查询和字段选择中，只有非虚拟属性起作用。由于虚拟属性不是储存在MongoDB中，所以你不可以用它们来查询。

##### 别名
别名是一种特殊类型的虚拟属性，他可以使getter和setter无缝的获取和设置另外的属性。这有助于节省网络带宽，因此你可以将存在数据库中的短属性名转换成长名字来提高代码可读性。

```js
var personSchema = new Schema({
  n: {
    type: String,
    // Now accessing `name` will get you the value of `n`, and setting `n` will set the value of `name`
    alias: 'name'
  }
});

// Setting `name` will propagate to `n`
var person = new Person({ name: 'Val' });
console.log(person); // { n: 'Val' }
console.log(person.toObject({ virtuals: true })); // { n: 'Val', name: 'Val' }
console.log(person.name); // "Val"

person.name = 'Not Val';
console.log(person); // { n: 'Not Val' }
```

你还可以在嵌套路径上声明别名。使用嵌套模式和子文档更容易，但是你也可以声明内联的嵌套路径别名，只要你用全部的嵌套路径`nexted.myProp`作为别名。

```js
const childSchema = new Schema({
  n: {
    type: String,
    alias: 'name'
  }
}, { _id: false });

const parentSchema = new Schema({
  // 如果在子schema中，别名不需要包含全部的嵌套路径
  c: childSchema,
  name: {
    f: {
      type: String,
      // 如果你声明为内联，别名需要包含全部的嵌套路径
      alias: 'name.first'
    }
  }
});
```

### Options
Schemas有一些可配置的选项，可以传递给构造函数或直接设置：

```js
new Schema({..}, options);

// or

var schema = new Schema({..});
schema.set(option, value);
```

可选选项：
- autoIndex
- bufferCommands
- capped
- collection
- id
- _id
- minimize
- read
- writeConcern
- safe
- shardKey
- strict
- strictQuery
- toJSON
- toObject
- typeKey
- validateBeforeSave
- versionKey
- collation
- skipVersioning
- timestamps

#### 选项:autoIndex 