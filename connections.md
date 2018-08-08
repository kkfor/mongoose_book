## 连接（Connections）
你可以用`mongoose.connect()`方法连接MongoDB。

```js
mongoos.connect('mongodb://localhost:27017/myapp');
```

这是连接运行在本地`myapp`数据库最小的值（27017）。如果连接失败，尝试用`127.0.0.1`代替`localhost`。  
你可在`uri`中指定更多的参数：
```js
mongoose.connect('mongodb://username:password@host:port/database?options...');
```
有关更多细节查看mongodb连接字符串规范。

### 操作缓冲
Mongoose可以让你立即使用模型，不用等待mongoose与MongoDB建立连接。

```js
mongoose.connect('mongodb://localhost:27017/myapp');
var MyModel = mongoose.model('Test', new Schema({ name: String }));
// Works
MyModel.findOne(function(error, result) { /* ... */ });
```

这是因为mongoose缓冲模型函数被立即调用。这种缓冲非常方便，但也是造成混淆的常见原因。默认情况下，如果你用一个没有连接的模型，Mongoose不会报错。

```js
var MyModel = mongoose.model('Test', new Schema({ name: String }));
// 将一直挂起，直到连接成功
MyModel.findOne(function(error, result) { /* ... */ });

setTimeout(function() {
  mongoose.connect('mongodb://localhost:27017/myapp');
}, 60000);
```

关闭schema的配置`bufferCommands`就可以禁用缓冲。如果在`bufferCommands`打开时有连接挂起，尝试关闭`bufferCommands`观察是否没有正确打开连接。你也可以全局禁用`bufferCommands`:

```js
mongoose.set('bufferCommands', false);
```

### 选项
`connect`方法也接收一个`options`对象，它将传递给底层的MongoDB驱动程序。

```js
mongoose.connect(uri, options);
```

可以在MongoDB Node.js驱动文档上找到`connect()`中选项的完整列表。Mongoose在没有修改的情况下将选项传递给驱动，除以下展示的例外。
- `bufferCommands` - 这是mongoose中一个特殊的选项（不传递给MongoDB驱动），它可以禁用mongoose的缓冲机制
- `user/pass` - 身份验证的用户名和密码。这是mongoose中特殊的选项，它们可以等同于MongoDB驱动中的auth.user和auth.password选项。
- `autoIndex` - 默认的，mongoose在连接的时候自动会在schema中创建索引。这对开发来说非常好，但是对大型项目部署并不理想，因为索引的构建将会造成性能的降低。如果设置`autoIndex`为false,mongoose在连接的时候不会为任何与此连接相关的模型创建索引。
- `dbName` - 指定连接哪个数据库，并覆盖连接字符串中任意的数据库。如果你用mongodb+srv语法连接MongoDB Atlas，你应该用`dbName`去指定数据库，因为当前不能再字符串中使用。

以下是调用mongoose中一些重要的选项。
- `useNewUrlParser` - 底层MongoDB已经废弃当前连接字符串解析器。因为这是一个重大的改变，添加了`useNewUrlParser`标记如果在用户遇到bug时，允许用户在新的解析器中返回旧的解析器。除非连接阻止设置，否则你应该设置`useNewUrlParser: true`。
- `autoReconnect` - 当与MongoDB连接断开时，底层MongoDB驱动将会自动尝试重新连接。除非你是高级用户，想要控制他们自己的连接池，否则不要设置这个选项为`false`。
- `reconnectTries` - 如果你连接到单个服务器或mongos代理（而不是副本集），MongoDB驱动将会在`reconnectTries`时间内的每一个`reconnectInterval`毫秒内重新连接，直到最后放弃连接。当驱动放弃连接的时候，mongoose连接将会触发`reconnectFailed`事件。此选项不会对副本集连接执行任何操作。
- `reconnectInterval` - 查看`reconnectTries`
- `promiseLibrary` - 设置底层驱动的promise库
- `poolSize` - MongoDB驱动将为这个连接保持的最大socket数量。默认情况下，`poolSize`是5。请记住在MongoDB 3.4中，MongoDB每个socket每次只允许一个操作，如果你在进行中发现一些缓慢的查询阻止快的查询，你可以增加这个值。
- `connectTimeoutMS` - MongoDB驱动在初始化连接失败时会等待多久。一旦Mongoose成功连接，`connectTimeoutMS`就不再有效。
- `socketTimeoutMS` - MongoDB驱动在杀掉一个不活跃的socket时会等待多久。socket可能因为不再活动或长时间处于操作状态而处于不活跃状态。默认情况这个值是`30000`，如果你希望一些数据库操作运行事件超过20秒，你应该设置为最长运行时间的2-3倍。
- `family` - 是否用IPv4或IPv6进行连接。这个选项传递个Node.js的`dns.lookup()`函数。如果你不设置这个选项，MongoDB驱动首先会尝试IPv6如果失败再尝试IPv4。如果`mongoose.connect(uri)`花费很长时间，尝试`mongoose.connect(uri, {family: 4})`

例子：

```js
const options = {
  useNewUrlParser: true,
  autoIndex: false, // 不创建索引
  reconnectTries: Number.MAX_VALUE, // 总是尝试重新连接
  reconnectInterval: 500, // 每500ms重新连接一次
  poolSize: 10, // 维护最多10个socket连接
  // 如果没有连接立即返回错误，而不是等待重新连接
  bufferMaxEntries: 0,
  connectTimeoutMS: 10000, // 10s后放弃重新连接
  socketTimeoutMS: 45000, // 在45s不活跃后关闭sockets
  family: 4 // 用IPv4, 跳过IPv6
};
mongoose.connect(uri, options);
```

有关`connectTimeoutMS`和`socketTimeoutMS`的更多信息，查阅此页面

### 回调
`connect()`函数也接收一个回调参数，其返回一个promise。

```js
mongoose.connect(uri, options, function(error) {
  // 检查错误，初始化连接。回调没有第二个参数。
});

// 或者用promise
mongoose.connect(uri, options).then(
  () => { /** ready to use. The `mongoose.connect()` promise resolves to undefined. */ },
  err => { /** handle initial connection error */ }
);
```

### 连接字符串选项
你还可以将连接字符串中的驱动选项指定为URI查询字符串中的部分参数。这只适用于传递给MongoDB驱动的选项。你不能在查询字符串中设置特定的Mongoose选项，类似`bufferCommands`

```js
mongoose.connect('mongodb://localhost:27017/test?connectTimeoutMS=1000&bufferCommands=false');
// 以上等同于:
mongoose.connect('mongodb://localhost:27017/test', {
  connectTimeoutMS: 1000
  // 注意mongoose将不会从查询字符串中提取`bufferCommands`
});
```

将选项放入查询字符串选项不利于阅读。但是你只需要单独设置URI，而不是分开对`socketTimeoutMS`，`connectTimeoutMS`等设置。最佳实践是在开发和生产中将不同的选项，类似`replicaSet`或`ssl`放到连接字符串中，保持不变的类似`connectTimeoutMS`或`poolsize`放到对象中。
MongoDB文档具有支持连接字符串选项的完整列表。

### keepAlive注意
对于长时间运行的应用，通常谨慎的做法是用毫秒数来激活`keepAlive`。没有它，一段时间后你或许会看到“连接关闭”的错误，这似乎是没有理由的。如果确实是这样，在阅读完本文后，你或许决定开启`keepAlive`：

```js
mongoose.connect(uri, { keepAlive: true, keepAliveInitialDelay: 300000 });
```

`keepAliveInitialDelay`是在socket上启动`keepAlive`时要等待的毫秒数。从mongoose 5.2.0开始`keepAlive`默认被启动。

### 副本集连接
要连接到一个副本集需要通过逗号分隔的主机列表而不是单个主机。

```js
mongoose.connect('mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]' [, options]);
```

例如：

```js
mongoose.connect('mongodb://user:pw@host1.com:27017,host2.com:27017,host3.com:27017/testdb');
```

连接一个单点副本集，指定`replicaSet`选项

```js
mongoose.connect('mongodb://host1:port1/?replicaSet=rsName');
```

### Multi-mongos支持
你可以连接多个mongos实例，以便于分片集群中的高可用行。在mongoose 5.x中你不需要传递任何选项去连接多个mongos。

```js
// 连接2个服务器
mongoose.connect('mongodb://mongosA:27501,mongosB:27501', cb);
```

### 多连接
到目前为止我们已经用mongoose的默认连接来连接上MongoDB。有时我们需要对mongo开放多个连接，每个具有不同的读/写设置，或者只是可能针对不同的数据库。在这些情况下，我们可以使用`mongoose.createConnection()`，他接受上面已经介绍过的所有参数，并返回一个新的连接给你。

```js
const conn = mongoose.createConnection('mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]', options);
```

这个连接对象用于创建和索引模型。模型总是作用于单个连接。
