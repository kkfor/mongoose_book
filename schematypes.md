## 模式类型（SchemaTypes）
SchemaTypes为查询和其他处理路径默认值，验证，getter，setter，字段选择默认值，以及字符串和数字的特殊字符。
在mongoose中有效的SchemaTypes。
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

### 例子

```js
var schema = new Schema({
  name:    String,
  binary:  Buffer,
  living:  Boolean,
  updated: { type: Date, default: Date.now },
  age:     { type: Number, min: 18, max: 65 },
  mixed:   Schema.Types.Mixed,
  _someId: Schema.Types.ObjectId,
  decimal: Schema.Types.Decimal128,
  array: [],
  ofString: [String],
  ofNumber: [Number],
  ofDates: [Date],
  ofBuffer: [Buffer],
  ofBoolean: [Boolean],
  ofMixed: [Schema.Types.Mixed],
  ofObjectId: [Schema.Types.ObjectId],
  ofArrays: [[]],
  ofArrayOfNumbers: [[Number]],
  nested: {
    stuff: { type: String, lowercase: true, trim: true }
  },
  map: Map,
  mapOfString: {
    type: Map,
    of: String
  }
})

// example use

var Thing = mongoose.model('Thing', schema);

var m = new Thing;
m.name = 'Statue of Liberty';
m.age = 125;
m.updated = new Date;
m.binary = new Buffer(0);
m.living = false;
m.mixed = { any: { thing: 'i want' } };
m.markModified('mixed');
m._someId = new mongoose.Types.ObjectId;
m.array.push(1);
m.ofString.push("strings!");
m.ofNumber.unshift(1,2,3,4);
m.ofDates.addToSet(new Date);
m.ofBuffer.pop();
m.ofMixed = [1, [], 'three', { four: 5 }];
m.nested.stuff = 'good';
m.map = new Map([['key', 'value']]);
m.save(callback);
```

### SchemaType选项
你可以用类型或对象的类型属性直接声明一个schema。

```js
var schema1 = new Schema({
  test: String // `test` is a path of type String
});

var schema2 = new Schema({
  test: { type: String } // `test` is a path of type string
});
```

除了type属性外，你可以为路径指定其他属性。例如，你想在保存前将字符串变小写：

```js
var schema2 = new Schema({
  test: {
    type: String,
    lowercase: true // `test`总是被转化为小写
  }
});
```

转化小写属性仅对字符串有效，某些选项适用于所有的schema类型，一些选项适用于特定的schema类型。
##### 所有的Schema类型
- required: 布尔值或函数，如果为true，则为此属性添加必须的验证。
- default: 任意类型或函数，为路径设置一个默认的值。如果值是一个函数，则函数的返回值用作默认值。
- select: 布尔值，指定查询的默认预测。
- validate: 函数，对属性添加验证函数。
- get: 函数，用`Object.defineProperty()`对属性声明一个getter。
- set: 函数，用`Object.defineProperty()`对属性声明一个setter。
- alias: 字符串，只对mongoose>=4.10.0有效。定义一个具有给定名称的虚拟属性，该名称可以获取/设置这个路径。

```js
var numberSchema = new Schema({
  integerOnly: {
    type: Number,
    get: v => Math.round(v),
    set: v => Math.round(v),
    alias: 'i'
  }
});

var Number = mongoose.model('Number', numberSchema);

var doc = new Number();
doc.integerOnly = 2.001;
doc.integerOnly; // 2
doc.i; // 2
doc.i = 3.001;
doc.integerOnly; // 3
doc.i; // 3
```

##### 索引
你可以用schema类型选项声明MongoDB的索引。
- index: 布尔值，是否在属性中定义一个索引。
- unique: 布尔值，是否在属性中定义一个唯一索引。
- sparse: 布尔值，是否在属性中定义一个稀疏索引。

```js
var schema2 = new Schema({
  test: {
    type: String,
    index: true,
    unique: true // 如果指定`unique`为true，则为唯一索引
    // 如果设置`unique`为true，则指定`index`是可选的
  }
});
```

##### 字符串
- lowercase: 布尔值，是否总是对值调用`toLowerCase()`
- uppercase: 布尔值，是否总是对值调用`toUpperCase()`
- trim: 布尔值，是否总是对值调用`trim()`
- match: 正则，创建一个验证器，验证值是否匹配给定的正则表达式
- enum: 数组，创建一个验证器，验证值是否是给定数组中的元素
- minlength: 数字，创建一个验证器，验证值的长度是不是大于给定的数
- maxlength: 数字，创建一个验证器，验证值的长度是不是小于给定的数

##### 数字
- min: 数字，创建一个验证器，验证值是否大于等于给定的最小值
- max: 数字，创建一个验证器，验证值是否小于等于给定的最大的值

##### 日期
- min: 日期
- max: 日期

### 使用说明
#### Dates
内建的日期方法没有挂到mongoose变化跟踪逻辑上，这意味着如果你在文档中使用Date方法，或用`setMonth()`一类的方法修改它，mongoose将不识别这种修改，`doc.save()`将不会保存这种修改。如果你必须用内建的方法修改Date类型，在保存前用`doc.markModified('pathToYourDate')`告诉mongoose改变。

```js
var Assignment = mongoose.model('Assignment', { dueDate: Date });
Assignment.findOne(function (err, doc) {
  doc.dueDate.setMonth(3);
  doc.save(callback); // 不会保存修改

  doc.markModified('dueDate');
  doc.save(callback); // 会保存修改
})
```

#### Mixed
一个“任何类型都来源”SchemaType，它的灵活性源于难以维护的权衡。可以通过Schema.Type.Mixed或通过传递空对象来获取mixed。以下是等同的：

```js
var Any = new Schema({ any: {} });
var Any = new Schema({ any: Object });
var Any = new Schema({ any: Schema.Types.Mixed });
```

由于它是无模式类型，因为可以修改值为任意类型，但是Mongoose没有自动检查和保存这些变化的功能。告诉Mongoose混合类型的值已经改变，调用文档的`markModified(path)`方法将路径传递给刚更改的Mixed类型。

```js
person.anything = { x: [3, 4, { y: "changed" }] };
person.markModified('anything');
person.save(); // 任何东西现在都会被保存
```

#### ObjectIds
指定ObjectId的类型，在声明中使用declaration。

```js
var mongoose = require('mongoose');
var ObjectId = mongoose.Schema.Types.ObjectId;
var Car = new Schema({ driver: ObjectId });
// or just Schema.ObjectId for backwards compatibility with v2
```

### 布尔值
布尔值在Mongoose中是普通的JavaScript布尔值。默认的，Mongoose转化以下值为`true`：
- true
- 'true'
- 1
- '1'
- 'yes'

Mongoose转化以下值为`false`：
- false
- 'false'
- 0
- '0'
- 'no'

其他值都会造成转化错误。你可以用`convertToTrue`和`convertToFalse`属性来修改Mongoose中哪些值转化为true或false，哪些是JavaScript集合。

```js
const M = mongoose.model('Test', new Schema({ b: Boolean }));
console.log(new M({ b: 'nay' }).b); // undefined

// Set { false, 'false', 0, '0', 'no' }
console.log(mongoose.Schema.Types.Boolean.convertToFalse);

mongoose.Schema.Types.Boolean.convertToFalse.add('nay');
console.log(new M({ b: 'nay' }).b); // false
```

#### 数组
Mongoose中Schematype和子文档支持数组。Schematype数组也成为原始数组，子文档数组也被称为文档数组。

```js
var ToySchema = new Schema({ name: String });
var ToyBoxSchema = new Schema({
  toys: [ToySchema],
  buffers: [Buffer],
  strings: [String],
  numbers: [Number]
  // ... etc
});
```

数组是特殊的，因为隐含具有一个默认值[]（空数组）。

```js
var ToyBox = mongoose.model('ToyBox', ToyBoxSchema);
console.log((new ToyBox()).toys); // []
```

要覆盖默认值，需要设置默认值为`undefined`

```js
var ToyBoxSchema = new Schema({
  toys: {
    type: [ToySchema],
    default: undefined
  }
});
```

注意：指定一个空数组是等同于`Mixed`。以下都为Mixed创建数组：

```js
var Empty1 = new Schema({ any: [] });
var Empty2 = new Schema({ any: Array });
var Empty3 = new Schema({ any: [Schema.Types.Mixed] });
var Empty4 = new Schema({ any: [{}] });
```

#### Maps

5.1.0新特性

一个`MongooseMap`是一个内置Map类的子类。在这些文档中我们将交替使用map和MongooseMap。在Mongoose中map是如何用任意键创建一个嵌套文档。

```js
const userSchema = new Schema({
  // `socialMediaHandles` is a map whose values are strings. A map's
  // keys are always strings. You specify the type of values using `of`.
  socialMediaHandles: {
    type: Map,
    of: String
  }
});

const User = mongoose.model('User', userSchema);
// Map { 'github' => 'vkarpov15', 'twitter' => '@code_barbarian' }
console.log(new User({
  socialMediaHandles: {
    github: 'vkarpov15',
    twitter: '@code_barbarian'
  }
}).socialMediaHandles);
```

以上的雷子没有显式的声明`github`或`twitter`作为路径，但是，由于`socialMediaHandles`是一个map，你可以储存任意键/值对。然而由于`socialMediaHandles`是一个map，所以你必须用`get()`获取一个键的值和用`.set()`设置一个键的值。

```js
const user = new User({
  socialMediaHandles: {}
});

// Good
user.socialMediaHandles.set('github', 'vkarpov15');
// Works too
user.set('socialMediaHandles.twitter', '@code_barbarian');
// Bad, the `myspace` property will **not** get saved
user.socialMediaHandles.myspace = 'fail';

// 'vkarpov15'
console.log(user.socialMediaHandles.get('github'));
// '@code_barbarian'
console.log(user.get('socialMediaHandles.twitter'));
// undefined
user.socialMediaHandles.github;

// Will only save the 'github' and 'twitter' properties
user.save();
```

在MongoDB中map类型储存为BSON对象。在一个BSON对象中键是有序的，这意味着map维护着插入顺序的属性。

### Getters
Getter在schema中是像一个虚拟属性为了路径声明。例如，你想储存用户个人资料图片作为相对路径，然后在你的应用中添加主机名。以下是构建userSchema的方法：

```js
const root = 'https://s3.amazonaws.com/mybucket';

const userSchema = new Schema({
  name: String,
  picture: {
    type: String,
    get: v => `${root}${v}`
  }
});

const User = mongoose.model('User', userSchema);

const doc = new User({ name: 'Val', picture: '/123.png' });
doc.picture; // 'https://s3.amazonaws.com/mybucket/123.png'
doc.toObject({ getters: false }).picture; // '123.png'
```

通常，你只在原始路径上使用getter而不是在数组或子文档中。因为getter会覆盖Mongoose路径返回的内容，在一个对象上声明一个getter可能会移除Mongoose对路径改变的跟踪。

```js
const schema = new Schema({
  arr: [{ url: String }]
});

const root = 'https://s3.amazonaws.com/mybucket';

// Bad, don't do this!
schema.path('arr').get(v => {
  return v.map(el => Object.assign(el, { url: root + el.url }));
});

// Later
doc.arr.push({ key: String });
doc.arr[0]; // 'undefined' because every `doc.arr` creates a new array!
```

如上所示，你应该声明一个getter在`url`字符串，而不是在数组上声明一个getter。如果你需要在嵌套文档或数组上声明getter，请小心。

```js
const schema = new Schema({
  arr: [{ url: String }]
});

const root = 'https://s3.amazonaws.com/mybucket';

// Good, do this instead of declaring a getter on `arr`
schema.path('arr.0.url').get(v => `${root}${v}`);
```

### 创建自定义类型
Mongoose可以扩展自定义SchemaType。在插件网站搜所兼容类型类似mongoose-long，mongoose-int32和其他类型。

### `schema.path()`函数
`schema.path()`返回给定路径的实例化模式类型。

```js
var sampleSchema = new Schema({ name: { type: String, required: true } });
console.log(sampleSchema.path('name'));
// Output looks like:
/**
 * SchemaString {
 *   enumValues: [],
 *   regExp: null,
 *   path: 'name',
 *   instance: 'String',
 *   validators: ...
 */
```

你可以用这个函数检查给出路径的模式类型，包含验证器和它的类型。

### 接下来
现在已经介绍完Schematype，让我们看一看Connections。