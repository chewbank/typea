### Install

      $ npm install check-data --save

### 使用方法

```js
let Validator = require('check-data')

let { error, data } = Validator(data, options, customize)
```

### 输入

*  `data` *Objcte, Array, String, Number, Date, Boolean* - 输入待验证数据

*  `options` *Objcte, Array, Function* - 数据验证表达式，类型参考type选项

*  `handler` *Objcte* - 自定义数据构建对象，根据输入数据生成新的数据结构（可选）

*  `handler.$` *Function* - 数据构建方法，函数名称与输出data对象中的key相对应。通过this或第一个函数参数可快速获取验证结果。(函数返回对象中同样支持多层嵌套函数表达式，可适应更多的应用场景。)

### 输出

*  `error` *String* - 验证失败时返回的错误信息，包含错误的具体位置信息，仅供开发者调试使用

*  `data` *Objcte* - 经过验证、处理后导出数据，内置空值过滤，自动剔除对象、数组中的空字符串、undefind值。（更多空值过滤特性请参考[filter-null模块](https://github.com/xiangle/filter-null)）

*  `msg` *String* - 验证失败后返回的错误信息，相对于error而言，msg对用户更加友好，可直接在客户端显示


### options

#### 通用选项

* `type` *String, Number, Object, Array, Date, Boolean, "MongoId", "MobilePhone", 'Email'* - 数据类型，扩展类型用字符串表示

* `name` *String* - 自定义参数名称，用于错误返回值中替换默认参数名

* `default` * - 默认赋值

* `value` * - 直接通过表达式赋值，类似于default选项，区别是不管值是否为空都将使用该值覆盖（优先级低于default，目前没有发现同时使用的应用场景）

* `allowNull` *Boolean* - 是否允许为空（默认将undefined和空字符串被视为空），缺省值为true。当值为false时，必须正确匹配指定的数据类型，否则会提示数据类型错误。

* `ignore` *Array* - 忽略指定的值，在字段级覆盖对默认空值的定义，如将某个指定字段的空值定义为[null, ""]

* `and` *Array、Function* - 声明依赖的参数名数组，支持数组和函数两种表达式，函数表达式用于声明指定值的依赖关系。要求依赖的所有参数都不能为空

* `or` *Array、Function* - 与and相似，区别是只要求依赖的其中一个参数不为空即可

* `handle` *Function* - 参数自定义转换方法，非空值时执行（原method方法更名为handle）。


#### String

* `minLength` *Number* - 限制字符串最小长度

* `maxLength` *Number* - 限制字符串最大长度

* `reg` *RegExp* - 正则表达式验证

* `in` *Array* - 匹配多个可选值中的一个

#### Number

* `min` *Number* - 限制最小值

* `max` *Number* - 限制最大值

* `in` *Array* - 匹配多个可选值中的一个

* `to` * - 类型转换，仅支持Boolean值

#### Array

* `minLength` *Number* - 限制字符串最小长度

* `maxLength` *Number* - 限制字符串最大长度

#### Object

> 仅支持类型验证

#### Date、Boolean

> 仅支持类型验证


### 其它类型

#### 'MongoId'

> 验证mongodb中的ObjectId

#### 'MobilePhone'

> 验证手机号

#### 'Email'

> 验证Email


### 扩展自定义数据类型

> 通过Validator.use()方法添加自定义的数据类型，使用方法和扩展类型一样，用字符串声明数据类型

```js
Validator.use(name, options)

# 示例
Validator.use('Int', {
   type({ data }) {
      if (Number.isInteger(data)) {
         return { data }
      } else {
         return { error: '必须为Int类型' }
      }
   },
})
```

### schema验证（预定义验证器）

> 通过预定义schema，实现options单例复用（option为静态数据），避免频繁创建重复的实例，可节省内存和减少计算开销。在环境允许的情况下应优先考虑schema方式。


```js
Validator.schema(name, options)
```

### 参考示例

#### schema验证

```js
let schema = Validator.schema('demo', {
   a: {
      a1: {
         type: Number,
         allowNull: false
      },
      a2: {
         type: Number,
         allowNull: false
      }
   },
   b: Number,
})

let json = {
   a: {
      a1: "jj",
      a2: "12",
   },
   b: 2,
   c: 888,
}

let { error, data } = schema(json)

// 或
let { error, data } = Validator.demo(json)
```

#### 数组验证

```js
// 一维数组
let { error, data } = Validator(["a", "b", "c"], [String])

// 内嵌对象
let { error, data } = Validator([{
   "a":1,
   "b":"bibi",
   "c":"test"
},{
   "a":1,
   "b":"bibi",
   "c":"test"
}], [{
   a: Number,
   b: String,
   c: String
}])
```

#### 对象验证

```js
let { error, data } = Validator({
   "a": 1,
   "b": "xx",
   "c": [1,32,34],
   "d": 666
}, {
   "a": Number,
   "b": String,
   "c": [String],
   "d": Number
})
```

#### and验证

```js
let { error, data } = Validator({
   "username": "莉莉",
   "addressee": "嘟嘟",
}, {
   "username": {
      "type": String,
      // 使用数组表达式
      "and": ["addressee", "address"],
   },
   "addressee": {
      "type": String,
      "allowNull": true
   },
   "address": {
      "type": String,
      "allowNull": true,
      // 使用函数表达式，表示特定值的依赖关系
      and(value){
         if (value === 1) {
            return ["addressee", "address"]
         } else if (value === 2) {
            return ["username", "xx"]
         }
      },
   }
})
```

#### or验证

```js
let { error, data } = Validator({
   "username": "莉莉",
   "addressee": "嘟嘟",
}, {
   "username": {
      "type": String,
      "or": ["addressee", "address"]
   },
   "addressee": {
      "type": String,
      "allowNull": true
   },
   "address": {
      "type": String,
      "allowNull": true
   }
})
```

#### 扩展类型验证

```js
let { error, data } = Validator({
   "id": "5968d3b4956fe04299ea5c18",
   "mobilePhone": "18555555555",
}, {
   "id": "MongoId",
   "mobilePhone": "MobilePhone"
})
```

#### 完整示例

```js
# 输入数据
let json = {
   "username": "测试",
   "num": "123456789987",
   "time": "2017-07-07T09:53:30.000Z",
   "files": ["abc.js", "334", "null", "666", , , "kkk.js"],
   "user": {
      "username": "莉莉",
      "age": 18,
   },
   "list": [
      {
         "username": "吖吖",
         "age": 16,
      },
      {
         "username": "可可",
         "age": 15,
      }
   ],
   "auth": {
         "weixin": "abc",
   },
   "beneficiariesName": "莉莉",
   "guaranteeMoney": 2,
   "guaranteeFormat": 0,
   "addressee": "嘟嘟",
   "receiveAddress": "北京市",
   "phone": "18666666666",
   "coupon": "uuuu",
   "integral": {
         "lala": 168,
         "kaka": "3"
   },
   "search": "深圳",
   "email": "xxx@xx.xx"
}

# 验证表达式
let { error, data } = Validator(json,
   {
      "username": {
         "type": String,
         "name": "用户名",
         "allowNull": false
      },
      "num": String,
      "time": {
         "type": Date,
         "name": "时间",
         "allowNull": false,
      },
      "user": {
         "username": String,
         "age": Number,
      },
      "list": [{
         "username": String,
         "age": Number,
      }],
      "auth": {
         "weixin": String,
      },
      "beneficiariesName": String,
      "guaranteeMoney": {
         "type": Number,
         "in": [1, 2]
      },
      "files": [{
         "type": String,
         "allowNull": false,
      }],
      "guaranteeFormat": {
         "type": Number,
         "conversion": Boolean
      },
      "addressee": {
         "type": String,
         "value": "直接通过表达式赋值"
      },
      "search": String,
      "phone": {
         "type": "MobilePhone"
      },
      "receiveAddress": String,
      "coupon": {
         "type": String,
         handle(value) {
            return { "$gt": new Date() }
         }
      },
      "integral": {
         "lala": {
            "type": Number,
         },
         "kaka": {
            "type": Number,
            "in": [1, 2, 3],
         }
      },
      "email": {
         "type": 'Email',
      },
   },
   {
      filter({ search, email, integral }) {
         return {
            "email": email,
            "integral": integral,
            "test": {
               a: 1,
               b: undefined,
               c: "",
               d: null,
               e: NaN,
               e: 0,
            },
         }
      }
   }
)
```
