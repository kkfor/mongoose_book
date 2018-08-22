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

