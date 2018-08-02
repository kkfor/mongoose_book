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

在`blogSchema`中的每个键都定义了文档中的一个属性。每个键都将转换成与它关联的SchemaType。例如，我们已经定义的属性`title`将被转换成`String`类型SchemaType，属性`date`将被转换成`Date`类型SchemaType。键也可以被分配包含深层的键/类型声明的嵌套对象，类似上面的meta属性。

允许的SchemaTypes有：
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

Schemas不仅定义文档的结构和属性类型，而且还定义文档的实例方法，静态模型方法，复合索引和叫做中间件的文档生命周期钩子。

### 创建一个模型
使用schema来定义，我们需要将`blogSchema`转换成一个可以工作的Model，所以，我们通过`mogoose.model(modelName, schema)`来转换：

```js
var Blog = mongoose.model('Blog', blogSchema);
// ready to go!
```

### 实例方法
Models实例是文档。文档有很多它们内部的实例方法。也可以声明自定义的文档实例方法。

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
应用启动时，Mongoose在`Schema`中对每个定义的索引发送一个`ceateIndex`命令。在Mongoose v3中，默认情况下，在后台创建索引。如果你希望在索引被创建的时候关闭自动创建功能并手动创建。设置`Schema`的`autoIndex`选项为`false`，并在model上使用ensureindexes方法。

```js
var schema = new Schema({..}, { autoIndex: false });
var Clock = mongoose.model('Clock', schema);
Clock.ensureIndexes(callback);
```

#### 选项：bufferCommands
默认情况下，mongoose在连接失败时会发出命令，直到驱动重新连接。禁用发出命令，设置`bufferCommands`为`fasle`。

```js
var schema = new Schema({..}, { bufferCommands: false });
```

schema的`bufferCommands`选项覆盖了全局的`bufferCommands`选项。

```js
mongoose.set('bufferCommands', true);
// 如果schema设置，下面的Schema选项会覆盖上面的。
var schema = new Schema({..}, { bufferCommands: false });
```

#### 选项：capped
Mongoose支持MongoDB的capped集合，要指定在MongoDB集合的`capped`，用字节设置`capped`选项改变集合的最大大小。

```js
new Schema({..}, { capped: 1024 });
```

如果你想设置额外类似`max`或`autoIndexId`的选项，可以设置`capped`为一个对象。在这种情况，你必须设置`size`选项。

```js
new Schema({..}, { capped: { size: 1024, max: 1000, autoIndexId: true } });
```

#### 选项：collection
默认情况下，Mongoose将通过传递模型名到utils.toCollectionName方法来生成集合名。改方法使名字更多样。设置此选项可在集合中得到不同的名字。

```js
var dataSchema = new Schema({..}, { collection: 'data' });
```

#### 选项：id
Mongoose默认会为每个schema分配一个`id`虚拟getter，它将文档的`_id`字段转化为字符串，或者在ObjectIds情况下，转化为hexString。如果你不想在schema中添加`id`getter，你可以在构建schema时，设置id选项来禁用。

```js
// 默认设置
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // '50341373e894ad16347efe01'

// 禁用id
var schema = new Schema({ name: String }, { id: false });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // undefined
```

#### 选项：_id
如果schema不传递禁Schema构造器，Mongoose默认会为schema分配一个`_id`字段。为了符合MongoDB的默认行为，`id`的类型是一个ObjectId。如果你不想在schema中添加`_id`，可以通过设置禁用它。  
只能在子文档中用这个选项。Mongoose不能保存没有id的文档，所以如果你尝试保存一个没有`_id`的文档将会报错。

```js
// 默认设置
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p); // { _id: '50341373e894ad16347efe01', name: 'mongodb.org' }

// 禁用_id
var childSchema = new Schema({ name: String }, { _id: false });
var parentSchema = new Schema({ children: [childSchema] });

var Model = mongoose.model('Model', parentSchema);

Model.create({ children: [{ name: 'Luke' }] }, function(error, doc) {
  // doc.children[0]._id will be undefined
});
```

#### 选项：minimize
Mongoose默认情况下通过删除空对象来最小化schema

```js
var schema = new Schema({ name: String, inventory: {} });
var Character = mongoose.model('Character', schema);

// 如果`inventory`字段不为空，将会保存它
var frodo = new Character({ name: 'Frodo', inventory: { ringOfPower: 1 }});
Character.findOne({ name: 'Frodo' }, function(err, character) {
  console.log(character); // { name: 'Frodo', inventory: { ringOfPower: 1 }}
});

// 如果`inventory`字段为空，将不会保存它
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam' }
});
```

通过设置`minimize`为`false`，这种行为将无效。Mongoose将会保存空对象。

```js
var schema = new Schema({ name: String, inventory: {} }, { minimize: false });
var Character = mongoose.model('Character', schema);

// 如果`inventory`字段为空也会保存它
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam', inventory: {}}
});
```

#### 选项：read
允许在schema级别设置查询#read,为我们提供一种默认应用ReadPreferences去从model中查询驱动。

```js
var schema = new Schema({..}, { read: 'primary' });            // also aliased as 'p'
var schema = new Schema({..}, { read: 'primaryPreferred' });   // aliased as 'pp'
var schema = new Schema({..}, { read: 'secondary' });          // aliased as 's'
var schema = new Schema({..}, { read: 'secondaryPreferred' }); // aliased as 'sp'
var schema = new Schema({..}, { read: 'nearest' });            // aliased as 'n'
```

每个pref的别名都是允许的，所以我们不用输入“secondaryPreferred”导致拼写错误，直接输入“sp”就可以。
read选项允许我们设置特殊的标记。它们告诉驱动应该尝试从哪个副本集成员去读。点击这里阅读更多关于标记设置。

注意：你还可以在连接是指定驱动读取pref策略选项：

```js
// 定时ping回复成员pings来跟踪网络延迟
var options = { replset: { strategy: 'ping' }};
mongoose.connect(uri, options);

var schema = new Schema({..}, { read: ['nearest', { disk: 'ssd' }] });
mongoose.model('JellyBean', schema);
```

#### 选项：writeConcern
允许在schema级别设置wirte concern

```js
const schema = new Schema({ name: String }, {
  writeConcern: {
    w: 'majority',
    j: true,
    wtimeout: 1000
  }
});
```

#### 选项：safe
`safe`是一个遗留类似`writeConcern`的选项。特别的是，在设置`writeConcern: { w:0 }`时，`safe: false`时无效的。改用`writeConcern`来代替。
#### 选项：shardKey
`shardKey`在分片MongoDB架构中是有用的。每个分片集合都有一个分片键，它在插入/更新操作中必须存在。我们只需要将schema选项设置为相同的分片键就可以了。

```js
new Schema({ .. }, { shardKey: { tag: 1, name: 1 }})
```

注意Mongoose不会发送`shardcollection`，你必须配置分片。

#### 选项：strict
strict选项默认被启用，确保传递到模型构造函数并且没有在schema中指定的值，不会保存到数据库中。

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema不会保存到数据库

// set to false..
var thingSchema = new Schema({..}, { strict: false });
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema现在会被保存到数据库
```

这也会影响使用`doc.set()`设置的属性值。

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.set('iAmNotInTheSchema', true);
thing.save(); // iAmNotInTheSchema不会保存到数据库
```

通过在model实例上传递第二个参数（布尔值），这个值将会被覆盖：

```js
var Thing = mongoose.model('Thing');
var thing = new Thing(doc, true);  // 启用strict模式
var thing = new Thing(doc, false); // 禁用strict模式
```

`strict`选项也可以被设置为`throw`，这将导致错误，而不是删除错误数据。
注意：无论schema选项是什么，在schema上不存在而在实例上存在的键/值总是会忽略的。

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.iAmNotInTheSchema = true;
thing.save(); // iAmNotInTheSchema永远不会保存到数据库
```

#### 选项：strictQuery
为了向后兼容，`strict`选项不能应用在查找的`filter`参数上。

```js
const mySchema = new Schema({ field: Number }, { strict: true });
const MyModel = mongoose.model('Test', mySchema);

// Mongoose不会过滤掉`notInSchema: 1`，尽管设置`strict: true`
MyModel.find({ notInSchema: 1 });
```

`strict`选项可以在update上应用

```js
// 如果设置了`strict`Mongoose将从更新中去掉`notInSchema`
// not `false`
MyModel.updateMany({}, { $set: { notInSchema: 1 } });
```

Mongoose有一个单独的`strictQuery`选项在查询的`filter`参数中切换严格模式。

```js
const mySchema = new Schema({ field: Number }, {
  strict: true,
  strictQuery: true // 打开严格模式
});
const MyModel = mongoose.model('Test', mySchema);

// 因为`stricQuery`设置为`true`，Mongoose将过滤掉`notInSchema: 1`
MyModel.find({ notInSchema: 1 });
```

#### 选项：toJSON
和toObject选项相同，但是只在文档调用toJSON时应用。
```js
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toJSON', { getters: true, virtuals: false });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m.toObject()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom' }
console.log(m.toJSON()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
// since we know toJSON is called whenever a js object is stringified:
console.log(JSON.stringify(m)); // { "_id": "504e0cd7dd992d9be2f20b6f", "name": "Max Headroom is my name" }
```

查看所有可用的`toJSON/toObject`选项，阅读这里。
#### 选项：toObject
文档有一个toObject方法，他可以将mongoose文档转换成一个普通的javascript对象。这个方法接收一些配置。我们可以声明这些配置，而不是在每个文档的基础之上应用这些，默认情况下它应用于所有schema文档。  
让所有虚拟属性在控制台输出，设置`toObject`选项`{getters: true}`

```js
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toObject', { getters: true });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```

查看所有可用的`toObject`选项，阅读这里。

#### 选项：typeKey
默认情况下，如果在schema中的对象有一个键为`type`，mongoose将解释它为类型声明。

```js
// Mongoose解释loc是一个字符串
var schema = new Schema({ loc: { type: String, coordinates: [Number] } });
```

然而，对于像geoJSON这样的应用，“type”属性非常重要。如果你想控制mongoose中用哪个键来查找类型声明，设置“typeKey”选项。

```js
var schema = new Schema({
  // Mongoose解释loc是一个包含两个键type和coordinates的对象
  loc: { type: String, coordinates: [Number] },
  // Mongoose解释name是一个字符串类型
  name: { $type: String }
}, { typeKey: '$type' }); // '$type'键表示该对象是类型声明
```

#### 选项：validateBeforeSave
默认情况下，文档在存入数据库时会被自动验证。无效的文档将不会被保存。如果你想手动控制验证，如果想不通过验证保存，可以设置`validateBeforeSave`为false。

```js
var schema = new Schema({ name: String });
schema.set('validateBeforeSave', false);
schema.path('name').validate(function (value) {
    return v != null;
});
var M = mongoose.model('Person', schema);
var m = new M({ name: null });
m.validate(function(err) {
    console.log(err); // null不允许，不能保存
});
m.save(); // 虽然无效，但成功保存
```

#### 选项：versionKey
Mongoose第一次创建每个文档的时候同时创建一个versionKey属性。这个属性值包含文档的内部修订版本。versionKey是代表版本控制的路径字符串。默认是`_v`。如果发生冲突，可以这样配置：

```js
var schema = new Schema({ name: 'string' });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { __v: 0, name: 'mongoose v3' }

// 自定义versionKey
new Schema({..}, { versionKey: '_somethingElse' })
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { _somethingElse: 0, name: 'mongoose v3' }
```

文档版本控制可以通过设置`versionKey`为`false`禁用。除非一定不需要，否则不要禁用。

```js
new Schema({..}, { versionKey: false });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'no versioning please' });
thing.save(); // { name: 'no versioning please' }
```

#### 选项：callation
为每个查询和聚合设置一个默认的collation，这是一个关于collation系统的概述。

```js
var schema = new Schema({
  name: String
}, { collation: { locale: 'en_US', strength: 1 } });

var MyModel = db.model('MyModel', schema);

MyModel.create([{ name: 'val' }, { name: 'Val' }]).
  then(function() {
    return MyModel.find({ name: 'val' });
  }).
  then(function(docs) {
    // `docs` will contain both docs, because `strength: 1` means
    // MongoDB会在匹配时忽略大小写
  });
```

#### 选项：skipVersioning
`skipVersioning`允许从版本控制中排除路径（例如：尽管路径更新了，内部修订也不会增加）。除非你需要，否则不要这样做。对于子文档，用全部的路径在父文档中包含。

```js
new Schema({..}, { skipVersioning: { dontVersionMe: true } });
thing.dontVersionMe.push('hey');
thing.save(); // version不会增加
```

#### 选项：timestamps
如果设置了`timestamps`，mongoose将为schema分配`createAt`和`updateAt`字段，分配字段的类型为`Date`。
默认情况，两个字段的名字为`createAt`和`updateAt`，通过设置timestamps.createAt和timestamps.updateAt来自定义字段名。
```js
var thingSchema = new Schema({..}, { timestamps: { createdAt: 'created_at' } });
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing();
thing.save(); // `created_at` & `updatedAt` will be included
```

#### 选项：useNestedString
在mongoose4，update()和findOneAndUpdate()只能检查顶级schema的严格模式设置。

```js
var childSchema = new Schema({}, { strict: false });
var parentSchema = new Schema({ child: childSchema }, { strict: 'throw' });
var Parent = mongoose.model('Parent', parentSchema);
Parent.update({}, { 'child.name': 'Luke Skywalker' }, function(error) {
  // Error because parentSchema has `strict: throw`, even though
  // `childSchema` has `strict: false`
});

var update = { 'child.name': 'Luke Skywalker' };
var opts = { strict: false };
Parent.update({}, update, opts, function(error) {
  // This works because passing `strict: false` to `update()` overwrites
  // the parent schema.
});
```

如果你设置`useNestedStrict`为true，mongoose将转换更新用子schema的strict选项

```js
var childSchema = new Schema({}, { strict: false });
var parentSchema = new Schema({ child: childSchema },
  { strict: 'throw', useNestedStrict: true });
var Parent = mongoose.model('Parent', parentSchema);
Parent.update({}, { 'child.name': 'Luke Skywalker' }, function(error) {
  // Works!
});
```

### Pluggable
模式是可插入的，它允许我们将可重用的特性打包成插件，这些插件可以被社区或你的其他项目间共享。
### 补充阅读
为获取更多的MongoDB知识，你需要学习MongoDB模式设计基础。SQL模式设计（第三范式）被设计为最小存储，而MongoDB模式设计是尽可能快的进行常见查询。MongoDB模式设计博客六法则是学习快速查询的基本规则的优秀资源。
希望在Node.js中掌握MongoDB模式设计的用户应该去看由MongoDB和Node.js的驱动者Christian Kvalheim写的的《The Little MongoDB Schema Design Book》。这本书向你展示了如果为一系列用例，包括电子商务，wiki和预约预定实现高性能的schema。
### 下一步
现在我们已经介绍完了`Schema`，接下来看一看`SchemaTypes`