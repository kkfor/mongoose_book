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