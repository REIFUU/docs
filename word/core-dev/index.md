# word-core[开发指南]

## 如何开发

先使用koishi官方的方案，导入word-core的同时使用它提供的服务

## word.cache [词库缓存相关]

词库每次启动的时候会建立一个缓存，存储着每个触发词所在的库

### WordCache

词库缓存

```
{
  hasKey: {
    [触发词]: [所在的词库]
  }
}
```

### word.cache.addCache(q, dbName)

标记某触发句存储于某词库

q：`string` 触发句

dbName: `string` 所存储的库

返回值：无

### word.cache.rmCache(q, dbName)

取消标记某触发句存储于某词库

q：`string` 触发句

dbName: `string` 所存储的库

返回值：无

### word.cache.cacheRefresh()

重建缓存

返回值：无

### word.cache.getCache()

获取当前的缓存

返回值：`WordCache` 词库缓存

### word.cache.nowCache

当前的缓存

返回值：`WordCache` 词库缓存

## word.config [词库核心配置相关]

词库核心的配置，配置一般为键值对

### word.config.getConfig(key)

获取配置

key：`string` 配置的键

返回值：`Promise<string | number | string[] | number[] | Record<string, string>>`

### word.config.updateConfig(key,value)

更新(新建)配置

key: `string` 配置的键

value: `string | number | string[] | number[] | Record<string, string>` 配置的值

返回值：`Promise<boolean>`

## word.driver [词库解释器相关]

词库通过此解释器来获取输入的`触发句`的`回复句`，并且解析`回复句`的内容

具体流程是：

词库的问答被设定后，是这样进行根据`触发句`，使得词库回复`回复句`的

```
ctx.on("message", session=>{
  const a = await ctx.word.driver.start(session,str=>{
    if (!str) {return;}
    session.send(str)
  })
})
```

`ctx.word.driver.start`做了以下的工作：

1. 进行词库搜索，若并未搜索到此`输入的句子`不为词库的`触发句`，则下一步
2. 进行输入替换后，再次搜索词库，查询此`输入的句子`是否为词库的`触发句`
3. 若此时未发现`输入`的为词库`触发句`，则`return;`
4. 若在此时发现输入的为词库的`触发句`，则寻找这个`触发句所在的词库`（若多个词库则随机选择一个）
5. 进入目标词库后，获取这个`触发句`的`回答句列表`
6. 随机挑选一个`回答句`进行解析，若成功解释则`ctx.word.driver.start` 的第二个参数的回调参数会获取到 `解释的结果` ，否则会收到`null`

### word.driver.start(session, str=>void)

session：`Session | {username:string , userId:string , channelId:string , content:string}` 

返回值：`Promise<string | null>` 解析结果

```
session.username    // 用户名
session.userId      // 用户id
session.channelId   // 频道
session.content     // 用户说的话
```

如果我们为词库设置了 `word.add 你好 你也好` 这样的问答

我们也可以使用

```javascript
ctx.word.driver.start({
  username: '你的名字',
  userId: '你的id',
  channelId: '你所在的频道',
  content: '你好'
},str=>{ console.log(str) })

// 此时str为"你也好"
```

## word.editor [词库编辑器相关]

词库编辑器是用于增改查删对话库的

每个库都是这样的结构，并且保存在`wordData`数据库内

```json
{
    "name": "此库的名字",
    "author": ["作者"],
    "data": {
        "触发句": [
            "回复句1",
            "回复句2"
        ]
    },
    "saveDB": "存储格"
}
```

### word.editor.addAuthor(name, uid, authorID)

添加一名新作者到当前某的库中

name：`string` 目标词库名

uid：`string` 你的id

authorID：`string` 被添加人的id

返回值：`Promise<"您不是作者，无权操作" | "添加完成" | "此作者已存在">`

### word.editor.removeAuthor(name, uid, authorID)

从某词库中删除一名作者

name：`string` 目标词库名

uid：`string` 你的id

authorID：`string` 被删除人的id

返回值：`Promise<"您不是作者，无权操作" | "此作者不存在" | "已删除作者">`

### word.editor.getAuthor(name)

获取某个词库的作者组

name：`string` 目标词库名

返回值：`Promise<string[]>` 作者组

### word.editor.isAuthor(name, authorID)

查看某人是否有作者权限

name：`string` 目标词库名

authorID：`string` 被查询者的id

返回值：`Promise<boolean>` 是否拥有

### word.editor.addWordItem(name, uid, q, a)

添加一条机器人问答

name：`string` 目标词库名

uid：`string` 你的id

q：`string` 触发句

a：`string` 回复句

返回值：`Promise<number | "您不是作者，无权操作">`

### word.editor.rmWordItem(name, uid, q, list)

删除一条机器人问答

name：`string` 目标词库名

uid：`string` 你的id

q：`string` 触发句

list：`number | "all"` 需要删除的词条对应的序号

返回值：`Promise<"您不是作者，无权操作" | "此触发词不存在" | "超过触发词的回答上限" | "over">`

### word.editor.getQuestion(q)

查找词条（查询某触发句所在的词库）

q：`string` 触发句

返回值：`Promise<string[]>` 包含此词条的数组

### word.editor.readWord(name)

获取某个词库的存储

name：`string` 词库名

返回值：

```json
{
    "name": "此库的名字",
    "author": ["作者"],
    "data": {
        "触发句": [
            "回复句1",
            "回复句2"
        ]
    },
    "saveDB": "存储格"
}
```

### word.editor.updateWord(name, data)

更新/写入词库

name：`string` 词库名

data：

```json
{
    "name": "此库的名字",
    "author": ["作者"],
    "data": {
        "触发句": [
            "回复句1",
            "回复句2"
        ]
    },
    "saveDB": "存储格"
}
```

### word.editor.readSaveCell(name, uid)

获取某词库的存储格

name：`string` 目标词库名

uid：`string` 你的id

返回结果：`Promise<string>`

### word.editor.setSaveCell(name, cell, uid)

设置某词库的存储格

name：`string` 目标词库名

cell：`string` 新存储格名

uid：`string` 你的id

返回值：`Promise<boolean | "你不是作者" | "无此词库">`

### word.editor.resetSaveCell(name, uid)

重设存储格到默认

name：`string` 目标词库名

uid：`string` 你的id

返回值：`Promise<boolean | "你不是作者" | "无此词库">`

### word.editor.getWordList()

获取当前所拥有的词库列表

返回值：`Promise<string[]>` 词库列表

### word.editor.removeWord(name)

删除词库到回收站

name：`string` 目标词库名

返回值：`Promise<"词库列表不存在此词库" | "ok">`

### word.editor.restoreWord(name)

恢复回收站的词库

name：`string` 目标词库名

返回值：`Promise<"回收站不存在此词库" | "ok">`

### word.editor.getRecycleBinList()

查看回收站存在的词库

返回值：`Promise<string[]>` 存在的词库列表

## word.permission [词库权限相关]

词库自带一套权限系统...(虽然很简陋

词库的权限系统为权限树形式的：

xxx.xxx.xxx这样的

### word.permission.add(uid, newPermission)

为某人添加权限

uid：`string` 被操作者uid

newPermission：`string` 权限树

返回值：`Promise<boolean | "已存在此权限">` 

### word.permission.rm(uid, permission)

删除某人的权限

uid：`string` 被操作者uid

permission：`string` 权限树

返回值：`Promise<boolean | "不存在此权限">` 

### word.permission.all(uid)

查看某人的全部权限

uid: `string` 查询目标uid

返回值：`Promise<string[]>` 所拥有权限列表

### word.permission.isHave(uid, permission)

判断某人是否拥有某权限

uid：`string` 查询目标uid

permission：`string` 权限树

返回值：`Promise<boolean>` 是否拥有

## word.statement [词库语法包相关]

词库的回复句中可以包括语法，`词库语法`是形如如下格式的字符串：

```
// 多参数语法
(语法名:参数1:参数2:...参数n)

// 单参数语法
(语法名:参数1)

// 无参数语法
(语法名)
```

比如如下情况，用户可以在回复句内加入一个无参数词库语法

```
word.add 你好 你也好(@this)！
```

我们希望，当你的名字是`测试员`时：

你发送`你好`的时候，bot会回复`你也好测试员！`

**我们该如何实现这一点？**

使用词库的api可以自定义语法包，像上述的情况，我们仅需要这样就可以实现：

```typescript
ctx.word.statement.addStatement('@this', async (inData, session) => {
  return session.username;
});
```

这段代码，定义了一个叫`@this`的语法包，当解释器在解释回复的时候，发现有`(@this)`这样的语法时，会调用此处的：

```typescript
async (inData, session) => {
  return session.username;
}
```

当调用此回调后，解释器会将`(@this)`替换为回调的返回值

### word.statement.addStatement(statementName, callback)

添加一个语法包

statementName：`string` 语法名

callback：`async function(inData, session) {}`

callback返回值: `string | void`

callbacl的`返回值`会替换掉`回复句`中正在解析中的`语法`

```typescript
// 此处的session是word.driver.start(session,ctx=>void)的session

// inData比较复杂，这个是一些方法和数据的封装，包含以下内容
inData.args
inData.matchs
inData.wordData
inData.parPack
inData.internal
```

#### inData.args

上面我们说到语法包有`多参`，`单参`，`无参`这三种。这些参数最终会保存到`inData.args`内，当然无参时则`inData.args = []`

举个例子：

```typescript
// 我们像这样添加一个词库问答
word.add 测试 (test:1233,4566,7899)

// 并且创建一个test语法包
ctx.word.statement.addStatement('test', async (inData, session) => {
  console.log(inData.args);
});

// 输出应该是：
// ["1233", "4566", "7899"]
```

#### inData.matchs

这个与后续的章节有关，我们会在下一章讲到

#### inData.wordData

此项为回复句所在的词库的信息

可以通过`inData.wordData`什么的获取此库的信息

#### inData.internal

拥有以下五项：

```typescript
// 更新某物品的数量
inData.internal.saveItem(uid, saveDB, itemName, itemNumber)
uid: string = 用户id
saveDB: string = 读取的存储格(可以从inData.wordData.saveDB中得到)
itemName: string = 物品名称
itemNumber: number = 物品数量

// 获取某物品的数量
inData.internal.getItem(uid, saveDB, itemName)
uid: string = 用户id
saveDB: string = 读取的存储格(可以从inData.wordData.saveDB中得到)
itemName: string = 物品名称

inData.internal.getUserConfig(uid, key)
获取用户的配置
uid: string = 用户id
key: string = 配置键

inData.internal.saveUserConfig(uid, key, value)
保存一个用户的配置
uid: string = 用户id
key: string = 配置键
value: settingTypeValue = 配置值

inData.internal.removeUserConfig(uid: string, key: string)
删除一个用户的配置
uid: string = 用户id
key: string = 配置键
```

请在获取物品的时候尽量使用这两个接口，而不是`word.user.getItem`和`word.user.updateItem`

`inData.internal`接口读取和更新用户数据的时候，会先建立个缓存，当所有语法包解析完成后，最终才会保存数据，而`word.user.getItem`和`word.user.updateItem`这些接口是立刻读取和立刻保存

#### inData.parPack

解析功能包，在语法包中return这些项的话可以在return时达成相应的效果

```typescript
return inData.parPack.next() // 放弃解释当前回复句，从当前可回复的回复句列表中再抽取一条进行解释
return inData.parPack.end()  // 放弃解释当前回复句，但是保存当前的用户数据
return inData.parPack.kill() // 放弃解释当前回复句，且不保存当前的用户数据
```

这里所谓的用户数据为上面`inData.internal`的api所操作用户数据

### word.statement.rmStatement(statementName)

删除一个语法包

statementName：`string` 语法名

### word.statement.statement

当前存在的语法包

## word.trigger [词库输入替换相关]

我们知道koishi的at的格式是这样的：

```
<at name="用户名" /> 或者 <at id="用户id" />这样的
```

我们想要在`at别人`的时候，让机器人回复`你好`，该如何制作呢

这时候我们就可以用到这里的输入替换功能啦

我们可以通过词库接口

```typescript
// 定义一个名为at的输入替换
ctx.word.trigger.addTrigger('at', '(@)', '\\s\*<at name=\\"([\\s\\S]+)\\"\\/>\\s\*');

// 在word.driver.start(session,str=>void)中，输入替换检测器会在
// session.content能够匹配到Reg('\\s\*<at name=\\"([\\s\\S]+)\\"\\/>\\s\*')时，将匹配的内容转换为(@)
// 这一步被称为输入替换

// 输入替换后，词库解释器才会接着运行
```

比如，我们设定这样的问答：

```
word.add (@) 你好呀！
```

有人发了一个`@测试员`，然后此信息被koishi收到后，会被转换为`<at name="测试员">`，于是此时`session.content = '<at name="测试员">'`

经过`输入替换`后，`session.content = '(@)'`

随后词库会进行解释器的工作。

### 注意

在替换之后！每次匹配到的结果，会存储在`inData.matchs`中

如上面的情况，`inData.matchs`为：

```
{
  at: ["测试员"]
}
```

### word.trigger.addTrigger(triggerName, replaceStr, matchReg)

添加一个输入替换

triggerName：`string` 替换名

replaceStr：`string` 替换结果

matchReg：`string` 正则字符串

注意！这里正则匹配的结果，会以`[triggerName]: [匹配结果]`放入到`inData.matchs`中

### word.trigger.rmTrigger(replaceStr, matchReg)

删除一个输入替换

triggerName：`string` 替换名

matchReg：`string` 正则字符串

### word.trigger.trigger

当前的输入替换列表

返回值：`{[key: string]: { reg: string[], id: string; }}`

## word.tools [词库工具]

词库工具包括操作数据库的基础api和一些简单的日常小工具

### 前提：

虽然使用了数据库，但是其实词库的数据库结构都是这样的类似键值对的存储方式

|id（主键）|data（值）|
|--|--|
|||

所以其实大概可以通过下面的接口修改成用json保存词库数据的方式？

<br/>

### word.tools.getDB(dbName)

获取一个数据库的内容

dbName: `string` 库的名字

返回值：

```typescript
Promise<{
    idList: string[];   // idList为主键数组
    dataList: any[];    // dataList为这个每个主键对应的值
}>
```

### word.tools.readDB(dbName, key)

获取数据库某个主键所在行的data的值

dbName: `string` 库的名字

key：`string` 查询的主键

返回值：`Promise<data>` 返回这个主键对应所在行的data的值

### word.tools.removeDB(dbName, key)

删除某个主键所在行

dbName: `string` 库的名字

key：`string` 所需要删除的主键

### word.tools.writeDB(dbName, key, data)

修改（或新建）某个主键的值

dbName: `string` 库的名字

key：`string` 查询的主键

data：`any` 主键所在行的data项的值

返回值：`Promise<boolean>` 成功/失败

### word.tools.randomNumber(min, max)

获取一个随机数

min：`number` 随机数下限

max：`number` 随机数上限

返回值：`number` 随机数

## word.user [词库用户相关] 

### word.user.getConfig(uid)

读取用户的配置，配置都为键值对格式存储

uid：`string` 用户id

返回值：`Promise<Record<string, string | number | string[] | number[] | Record<string, string>>>`

### word.user.setConfig(uid, key, value)

设置用户的配置到缓存，配置都为键值对格式存储

uid：`string` 用户id

key：`string` 配置的键

value：`string` 配置的值

### word.user.saveConfig()

保存设置的缓存

返回值：`Promise<boolean>` 成功/失败

### word.user.setConfigForce(uid, key, value)

设置用户的配置，并直接保存到数据库，配置都为键值对格式存储

uid：`string` 用户id

key：`string` 配置的键

value：`string` 配置的值

### word.user.getData(uid)

获取用户背包

uid：`string` 用户id

返回值：`Promise<Record<string, Record<string, number>>>` 背包数据

用户背包内容一般为：

```typescript
{
  [存储格名:string]: {
    [物品名称:string]: 数量:number
  }
}
```

### word.user.getEditWord(uid)

获取用户正在编辑的词库

uid：`string` 用户id

返回值：`Promise<string>` 正在编辑的词库

### word.user.getItem(uid, cell, itemName)

从数据库中读取用户数据

uid：`string` 用户id

cell：`string` 存储格

itemName：`string` 物品名称

返回值：`Promise<number | null>` 存在的数量数量或者null

### word.user.setEditWord(uid, newDB)

设置用户正在编辑的库

uid: `string` 用户id

newDB：`string` 需要编辑的库

### word.user.updateItem(uid, cell, itemName, amount)

为用户某物品的数量

uid：`string` 用户id

cell：`string` 存储格

itemName：`string` 物品名称

amount：`number` 物品数量

### word.user.updateData(uid, data)

保存用户数据到数据库

uid：`string` 用户id

data：`Record<string, Record<string, number>>`

data的格式必须为：

```typescript
{
  [存储格名:string]: {
    [物品名称:string]: 数量:number
  }
} 
```

### word.user.saveTemp()

保存缓存中的信息到数据库
