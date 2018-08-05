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
- lowercase: 布尔值，