title: Mongoose API
tags: []
categories: []
date: 2016-07-27 12:28:00
---
# **Mongoose API**

标签（空格分隔）： mongodb 



---
## 快速入门
### 入门
*首先确认你安装了 `MongoDB` 和 `Node.js`*。
然后用 `npm` 从命令行安装 `mongoose`：
```
$ npm install mongoose
```
现在说我们喜欢毛茸茸的小猫，希望在 `MongoDB` 中记录每一个遇到的小猫。第一件需要做的就是在项目中引用 `mongoose`，然后在本地的 `MongoDB` 的实例中的test数据库打开一个连接。
```javascript
// getting-started.js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');
```
我们有了一个运行在本地、连接被挂起的test服务器。现在我们可以得知服务器是否连接成功或者报错：
```javascript
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function (callback) {
  // yay!
});
```
<!-- more -->
一旦打开连接，callback将会被触发。为简洁起见，我们假设接下来的代码都不带有此callback。

在 Mongoose 中, 一切源于 **`Schema`**。下面让我们来引用他并定义一个kittens:
```javascript
var kittySchema = mongoose.Schema({
    name: String
})
```
我们得到了有一个字段的 `Schema` ,类型为 String , 字段名为  name。下面由该 `Schema` 发布一个 `model` 。
```javascript
var Kitten = mongoose.model('Kitten', kittySchema)
```
`model` 是一个用来创建`document`的 类。这样的话，每个`document`将会是一个具有抽象属性和行为的数据库操作的 `Kitten` 对象。下面创建一个`Kitten`的 `document` 实例来标识刚刚散步遇到的小猫吧：
```javascript
var silence = new Kitten({ name: 'Silence' })
console.log(silence.name) // 'Silence'
```
小猫可以喵喵叫，下面展示了如何为 `document` 添加 `speak`方法。
```javascript
// NOTE: methods must be added to the Schema before compiling it with mongoose.model()
kittySchema.methods.speak = function () {
  var greeting = this.name
    ? "Meow name is " + this.name
    : "I don't have a name"
  console.log(greeting);
}
var Kitten = mongoose.model('Kitten', kittySchema)
```
添加到 `Schema` 的 `method`属性中的方法，将被编译到到 `model` 原型中去并暴露在 `document` 实例：
```javascript
var fluffy = new Kitten({ name: 'fluffy' });
fluffy.speak() // "Meow name is fluffy"
```
现在有了可以喵喵叫的小猫！但是我们尚未在 MongoDB 中储存任何数据。每个 `document` 实例可以调用 自身的**`save`**方法存入数据库。`callback` 的第一个参数是发生错误时的返回码：
```javascript
fluffy.save(function (err, fluffy) {
  if (err) return console.error(err);
  fluffy.speak();
});
```
假设我们想要显示之前遇到的所有小猫，我们可以通过 `Kitten`找到所有的kitten `documents`：
```javascript
Kitten.find(function (err, kittens) {
  if (err) return console.error(err);
  console.log(kittens)
})
```
刚才我们在命令行打印了数据库中所有的kitten。如果我们想要通过name查询kitten, Mongoose 支持 MongoDB 的*富参数查询*：
```javascript
Kitten.find({ name: /^Fluff/ }, callback)
```
查询 `name` 字段以"Fluff"开头的 `doucment` 实例，在    `callback` 中返回结果。

###结束
快速入门已结束。我们创建了一个 `Schema`，添加了一个 `document` 方法，用 Mongoose 在 MongoDB 中保存和查询 kitten。更多请查看 guide 和 api docs。


----------

## **导航**
### **Schemas**
请确保你已看过快速入门章节再阅读本章节。
#### **定义Schema**
Mongoose中一切由Schema导出。每个Schema映射到MongoDB中的集合，并以集合定义骨架文本。
```javascript
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
每个 `blogSchema` 中的键定义的属性将会是 `SchemaType`定义的一个类型。比如，`title` 属性将会被定义成String, `date`将会被定义成data。也可以支持复杂的嵌套结构（比如上面的 `meta`）。

基本数据类型有：

- String
- Number
- Date
- Buffer
- Boolean
- Mixed
- ObjectId
- Array

`Schemas` 不仅定义文档结构和数据类型，也可以定义实例方法，静态方法，索引，被称为中间件的文档生命周期钩子等。 

#### **创建Model**
为了使用schema定义，需要将 `blogSchema` 转化成易于工作的 `Model` 。为此，使用 `mongoose.model(modelName, schema)`方法。  
```javascript
var Blog = mongoose.model('Blog', blogSchema);
// ready to go!
```
#### **实例方法(Instance methods)**
`Model`的实例都是 `document` , `document`有其自建的实例方法。也可以自己定义文档实例方法。
```javascript
// define a schema
var animalSchema = new Schema({ name: String, type: String });
// assign a function to the "methods" object of our animalSchema
animalSchema.methods.findSimilarTypes = function (cb) {
  return this.model('Animal').find({ type: this.type }, cb);
}
```
现在所有的Animal实例*都*有findSimilarTypes方法：
```javascript
var Animal = mongoose.model('Animal', animalSchema);
var dog = new Animal({ type: 'dog' });
dog.findSimilarTypes(function (err, dogs) {
  console.log(dogs); // woof
});
```
重写默认的实例文档方法可能导致*不可预知*的结果。

#### **静态方法(Statics)**
为 `model` 添加静态方法也同样简单：
```javascript
// assign a function to the "statics" object of our animalSchema
animalSchema.statics.findByName = function (name, cb) {
  this.find({ name: new RegExp(name, 'i') }, cb);
}
var Animal = mongoose.model('Animal', animalSchema);
Animal.findByName('fido', function (err, animals) {
  console.log(animals);
});
```
#### **索引（Indexes）**
MongoDB支持*二级索引*,在path层或者schema层定义索引。当创建复合索引时，在schema层定义索引很有必要。
```javascript
var animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true } // field level
});
animalSchema.index({ name: 1, type: -1 }); // schema level
```
当应用启动时，Mongoose为每个schema中的索引自动调用 `ensureIndex` 。为更好的开发，推荐将这项行为禁用因为索引可能导致显著的性能影响。在schema中设置 `autoIndex: false` 即可。
```javascript
animalSchema.set('autoIndex', false);
// or
new Schema({..}, { autoIndex: false });
```
#### **虚拟属性（Virtuals）**
虚拟属性可以在实例文档中get、set，但不能写入数据库。getters有助于格式化、连接字段，settings有助于将一个字段去结构化成多个字段存储。
```javascript
// define a schema
var personSchema = new Schema({
  name: {
    first: String,
    last: String
  }
});
// compile our model
var Person = mongoose.model('Person', personSchema);
// create a document
var bad = new Person({
    name: { first: 'Walter', last: 'White' }
});
```
假如要输出 `bad` 的全名，可以手动如下操作：
```javascript
console.log(bad.name.first + ' ' + bad.name.last); 
// Walter White
```
也可以在personSchema定义一个 `virtual property getter`,这样就不用每次使用时做字符串拼接了。
```javascript
personSchema.virtual('name.full').get(function () {
  return this.name.first + ' ' + this.name.last;
});
```
此时，当使用 `name.full` 属性时，将会调用getter返回结果。
```javascript
console.log('%s is insane', bad.name.full); 
// Walter White is insane
```
注意如果结果集被转化成object或者JSON,virtuals默认**不**被启用，在 `toObject()`、`toJSON()`中传入 `virtuals : true` 以启用。

也可以通过设置 `this.name.full` 来设置 `this.name.first` 和 `this.name.last`。比如，要分别设置 `bad.name.first`和 `bad.name.last` 为`"Breaking"` and `"Bad"` , 可做如下操作：
也可以在personSchema定义 `virtual property setters`：
```javascript
bad.name.full = 'Breaking Bad';
```javascript
personSchema.virtual('name.full').set(function (name) {
  var split = name.split(' ');
  this.name.first = split[0];## 标题 ##
  this.name.last = split[1];
});
...
mad.name.full = 'Breaking Bad';
console.log(mad.name.first); // Breaking
console.log(mad.name.last);  // Bad
```
虚拟属性setters在数据验证之前应用，所以上面的例子在 `first`、 `last` name字段即使是必须的也能正常使用。

只有非虚拟属性可以作为查询和字段选择。

#### **可配置项（Options）**
schema有一系列的可被直接传入构造器或设置的可配置选项。
```javascipt
new Schema({..}, options);
// or
var schema = new Schema({..});
schema.set(option, value);
```
有效选项:

- autoIndex
- capped
- collection
- id
- _id
- read
- safe
- shardKey
- strict
- toJSON
- toObject
- versionKey

**option: autoIndex**
当应用启动时，Mongoose给每个在schema定义索引发送一个 `ensureIndex` 命令。从mongoose V3开始，索引在 `background` 默认被创建。如果你希望禁用这项自动创建的特性并想在索引被创建时手动处理，将 `Schemas autoIndex` option设置为false然后在 `model` 中使用 `ensureIndex`。
```javascript
var schema = new Schema({..}, { autoIndex: false });
var Clock = mongoose.model('Clock', schema);
Clock.ensureIndexes(callback);
```
**option: bufferCommands**
当应用运行时 `autoReconnect`被禁用并连接到一个单个mongo实例， 此时若连接中断，`mongoose buffers` 将一直发送commands 直到你手动恢复。在如上情况下禁用 `mongoose buffer`，将其设置为false。
```javascript
var schema = new Schema({..}, { bufferCommands: false });
```
**option:capped**
mongoose支持有上限集合。为了指定这种有上限的潜在的集合,以bytwa为单位设置最大上限。
```javascript
new Schema({..}, { capped: 1024 });
```
如果需要传入额外的参数如 `max`, `autoIndexId`, 上限参数也可以被设置为对象,这种情况下, `size` 参数是必选的。
```javascript
new Schema({..}, { capped: { size: 1024, max: 1000, autoIndexId: true } });
```
**option: collection**
Mongoose默认将model name传入 `utils.toCollectionName` 以初始化集合。此方法将使model name变为复数。如需要非默认名称可以进行单独设置。
```javascript
var dataSchema = new Schema({..}, { collection: 'data' });
```
**option: id**
Mongoose默认地给每个schemas分配一个返回string类型或 `ObjectId` 形式 的`virtual getter` id，返回 `_id` 字段。如果不需要在schema中加入id getter，可以在schema创建时禁用。
```javascript
// default behavior
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // '50341373e894ad16347efe01'
// disabled id
var schema = new Schema({ name: String }, { id: false });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // undefined
```
**option: _id**
若Schema构造时没有初始化，Mongoose将默认地给每个schema分配一个`_id`字段， 类型为`ObjectId`类型，与mongoDB默认行为一致。如果不需要在schema中加入id getter，可以在schema初始化时禁用。
请在schema创建时传入该option以阻止Mongoose创建`_id`。
```javascript
// default behavior
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p); // { _id: '50341373e894ad16347efe01', name: 'mongodb.org' }
// disabled _id
var schema = new Schema({ name: String }, { _id: false });
// Don't set _id to false after schema construction as in
// var schema = new Schema({ name: String });
// schema.set('_id', false);
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p); // { name: 'mongodb.org' }
// MongoDB will create the _id when inserted
p.save(function (err) {
  if (err) return handleError(err);
  Page.findById(p, function (err, doc) {
    if (err) return handleError(err);
    console.log(doc); // { name: 'mongodb.org', _id: '50341373e894ad16347efe12' }
  })
})
```
**option: read**
*这个看不懂*

**option: safe**
若有错误返回callback,此参数将会被传入。
```javascript
var safe = { w: "majority", wtimeout: 10000 };
new Schema({ .. }, { safe: safe });
```
`safe`字段默认被设置为true,以保证任何产生的错误都能返回到callback。`safe` 可如下设置 `{ j: 1, w: 2, wtimeout: 10000 }` ,设置j表示做1份日志，w表示做2个副本（尚不明确），超时时间10秒。
```javascript
var safe = { w: "majority", wtimeout: 10000 };
new Schema({ .. }, { safe: safe });
```

**option: shardKey**
需要mongoose做分布式处理才会用到该属性。每个分布的集合分配一个必须在insert/update操作时传输的分布秘钥。只需设置schema option为同一个分布秘钥即可。
```javascript
new Schema({ .. }, { shardKey: { tag: 1, name: 1 }})
```
**option: strict**
默认开启strict option,用于保证传入model构造器中不准确的值不会被写入数据库。
```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema is not saved to the db
// set to false..
var thingSchema = new Schema({..}, { strict: false });
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema is now saved to the db!!
```
这同样影响 `doc.set()` 方法。
```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.set('iAmNotInTheSchema', true);
thing.save(); // iAmNotInTheSchema is not saved to the db
```
该参数可设置boolean值在model实例层重写。
```javascript
var Thing = mongoose.model('Thing');
var thing = new Thing(doc, true);  // enables strict mode
var thing = new Thing(doc, false); // disables strict mode
```
该参数也可被设为`"throw"`,此时传入错误的数据将会报错。
*注意：**不要**设置为false,除非有适当的理由。*
*注意：任何在实例中的设置的不存在于schema中的key/val将会被忽略，无视该参数。*
```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.iAmNotInTheSchema = true;
thing.save(); // iAmNotInTheSchema is never saved to the db
```
**option: toJSON**
和 `toObject`option 类似，但只在document.toString 被调用时起作用。
```javascript
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
**option: toObject**
`document.toObject` 可以将mongoose document转化为JavaScript对象。
该方法接受一系列参数。可以将参数定义在schema以默认地在schema产生的document中接受这些参数，而不是给每个document传入参数。
设置 `getters: true` 以使 `viturals` 也能在 `console.log` 中输出。
```javascript
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toObject', { getters: true });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```
**option: versionKey**
每个doucment被创建时会写入`versionKey`。此值包括该document内置的修订。该document属性的name是可配置的。默认的是 `_v`。如果与你的应用冲突了，你可以如下配置：
```javascript
var schema = new Schema({ name: 'string' });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { __v: 0, name: 'mongoose v3' }
// customized versionKey
new Schema({..}, { versionKey: '_somethingElse' })
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { _somethingElse: 0, name: 'mongoose v3' }
```
document版本控制也可设置为 `false` 禁用,***不要*** 禁用该option， *除非你知道你在做什么*。


----------


### **Models**
Models是由schema定义编译的假想的构造器，document表示Models的实例，可被存入数据库或从数据库恢复。所有数据库操作都是由model处理。
####  **编译model**
```javascript
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```
#### **生成document**
document是model的实例。创建doc并存入数据库是简单的：
```javascript
var Tank = mongoose.model('Tank', yourSchema);
var small = new Tank({ size: 'small' });
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
})
// or
Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
})
```
注意未打开连接的话任何tanks将不会被创建/删除。这种情况下，使用 `mongoose.model()` ,现在打开连接：
```javascript
mongoose.connect('localhost', 'gettingstarted');
```
**查询**
用Mongoose查询很简单，mongoose支持MongoDB的富参数查询。可以使用`models`静态方法 `find` , `findById` , `findOne` , or  `where` 。
```javascript
Tank.find({ size: 'small' }).where('createdDate').gt(oneYearAgo).exec(callback);
```
**删除**
`model.remove` 静态方法可删除满足`contitions`的所有文档。
```javascript
Tank.remove({ size: 'large' }, function (err) {
  if (err) return handleError(err);
  // removed!
});
```
**更新**
每个model有其自身的 `update` 方法用于在数据库中修改文档，无需将其返回你的应用。


----------
### **Documents**
Mongoose documents是存储于MongoDB中的点对点映射。每个document都是model的一个实例。
**查询**
有很多方法可以从MongoDB检索文档。详情见querying章节。

**更新**
有很多种方式可以更新docments。下面是一种用 `findById` 的传统的方式：
```javascript
Tank.findById(id, function (err, tank) {
  if (err) return handleError(err);
  
  tank.size = 'large';
  tank.save(function (err) {
    if (err) return handleError(err);
    res.send(tank);
  });
});
```
这个方法找到了mongo中第一条document，然后分配一个 `save` 命令。但是，如果我们不需要返回在应用中返回document，而是直接在数据库中直接更新属性。可以使用 `model.uopdate()` 。
```javascript
Tank.update({ _id: id }, { $set: { size: 'large' }}, callback);
```
如果需要返回document，可以使用：
```javascript
Tank.findByIdAndUpdate(id, { $set: { size: 'large' }}, function (err, tank) {
  if (err) return handleError(err);
  res.send(tank);
});
```
**验证**
文档在存储之前验证。
`findAndUpdate/Remove` 静态方法对至少一个document进行操作，然后只调用一次返回结果。


----------


### **Sub Docs**
嵌套文档是由已有schema创建的文档的集合。
```javascript
var childSchema = new Schema({ name: 'string' });
var parentSchema = new Schema({
  children: [childSchema]
})
```
嵌套文档和普通文档有着相同的特性。唯一的不同是嵌套文档不是单独的被存储，而是父元素存储时才被存储。
```javascript
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';
parent.save(callback);
```
如果在嵌套文档的中间件发生了错误，它将会冒泡到父元素的callback,此时error handling 被中断了。
```javascript
childSchema.pre('save', function (next) {
  if ('invalid' == this.name) return next(new Error('#sadpanda'));
  next();
});
var parent = new Parent({ children: [{ name: 'invalid' }] });
parent.save(function (err) {
  console.log(err.message) // #sadpanda
})
```
**找到嵌套文档**
每个文档有一个 `_id`。文档集合有特殊的id方法用于查找他的 `_id` 。
```javascript
var doc = parent.children.id(id);
```
**添加嵌套文档**
MongooseArray方法， `push` 、 `unshift` 、`addToSet`等显式地向合适的类型中注入参数。
```javascript
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
嵌套文档也可以通过MongooseArrays的 `create` 方法进行创建。
**删除嵌套文档**
每个嵌套文档有自身的 `remove` 方法。
```javascript
var doc = parent.children.id(id).remove();
parent.save(function (err) {
  if (err) return handleError(err);
  console.log('the sub-doc was removed')
});
```


----------


### **Queries**
文档可被models的若干静态方法查询。
任何带有准确查询条件的model方法有两种执行方式：
当callback：

- 被传入时，当结果返回callback时方法将会被立即执行。
- 未传入时，会返回一个提供特殊接口 `QueryBuilder` 的 `Query` 实例。

看看传入callback时发生了什么：
```javascript
var Person = mongoose.model('Person', yourSchema);
// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
})
```
现在可知查询被立刻执行，然后结果返回callback。mongoose中的callback使用如下参数 callback(error, result)。如果查询时发生错误， `error` 参数将包含一个错误文档，`result` 将会是null。如果查询成功， `error` 将会是null,而 `result` 将会是查询的结果的填充。

只要是在queryz中传入了callback, callback就遵循如下模式 `callback(error, result)`。  `result`取决于方法： `findOne` 是单个潜在的空文档， `find()` 是文档集合， `count` 是文档数量， `update` 是影响的文档数。

看看未传入callback时发生了什么：
```javascript
// find each person with a last name matching 'Ghost'
var query = Person.findOne({ 'name.last': 'Ghost' });
// selecting the `name` and `occupation` fields
query.select('name occupation');
// execute the query at a later time
query.exec(function (err, person) {
  if (err) return handleError(err);
  console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
})
```
返回的 `Query` 允许进行链式查询。深入的例子：
```javascript
Person
.find({ occupation: /host/ })
.where('name.last').equals('Ghost')
.where('age').gt(17).lt(66)
.where('likes').in(['vaporizing', 'talking'])
.limit(10)
.sort('-occupation')
.select('name occupation')
.exec(callback);
```