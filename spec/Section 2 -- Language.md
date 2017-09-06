# 语言

客户端使用GraphQL查询语言向GraphQL服务发出请求。我们将这些请求源称为文档（documents）。文档可能包含操作（查询（queries），变更（mutations）和订阅（subscriptions））以及片段（fragments），这（fragments）是允许查询重用的组成部分。

GraphQL文档被定义为句法语法，其中最小符号是token（不可分割的词汇单位）。 这些token在词法语法中定义，它由源字符（由双冒号`::`)定义。


## 源文本(Source Text)

SourceCharacter :: /[\u0009\u000A\u000D\u0020-\uFFFF]/

GraphQL文档表示为一个Unicode字符序列。然而，除了少数例外，GraphQL的大部分仅以原始的非控制ASCII(可参考：http://unicode.org/standard/standard.html)范围表示，以便尽可能广泛地兼容现有的工具，语言和序列化格式，并避免文本编辑器和源代码控制系统中的显示问题。


### Unicode

UnicodeBOM :: "字节顺序标记(Byte Order Mark) (U+FEFF)"

非ASCII Unicode字符可以自由出现在{StringValue}和
GraphQL的{Comment}部分。

字节顺序标记(Byte Order Mark)是一个特殊的Unicode字符。可能出现在文件的开头，以用来帮助程序确定文本流为Unicode和特别的Unicode的编码版本。


### 空白(WhiteSpace)

空白(WhiteSpace) ::
  - "水平制表符Tab (U+0009)"
  - "空格 (U+0020)"

空白用于提高源代码的可读性。任意数量的空白可能出现在token之前或者之后。
token之间的空白对于语法意义上来说并不重要，但是空白字符可能出现在
{String}或{Comment}token之中。

注意：GraphQL有意地不考虑Unicode“Zs”类别字符作为空白，避免来自于文本编辑和版本控制工具的误解。


### 换行(LineTerminator)

换行(LineTerminator) ::
  - "换行符 LF (U+000A)" (LF)
  - "回车符CR  (U+000D)" (和换行符(U+000A)不一样) (CR)
  - "回车符 CR (U+000D)"" 换行符 LF(U+000A)" (CRLF)

和空格一样，换行用于提高源代码的可读性，任何数量的换行符可能
出现在任何其他token之前或之后，并没有任何对于GraphQL文档的语法上的意义。
任何其他的token中都不存在终止符。

注意：任何提供行号的错误报告中关于非法语法的提示应该使用前面的
{LineTerminator}数量来生成


### 注释 (Comments)

注释 : `#` 

注释符 :: 除换行外所有的原始字符

GraphQL 的源码文档有时会包含{`#`}开头的单行注释。

一个注释可以包含除了 {LineTerminator} 之外的任何字符。所以一个注释总是会包含从 {`#`} 开始直到换行之前的所有字符

注释与空白相同，可能出现在任何token之后或换行之前，同时不对语句文段产生任何语义上的影响.

### 不重要的逗号(Insignificant Commas)

逗号(Comma) :: `,`

与空格和行终止符类似, 逗号(Comma)  ({`,`}) 用于提高源代码的可读性，并进行单独的词汇标记。除非是在GraphQL查询文档（query documents）之中，在语法和语意上都没有用处。

不重要的逗号字符确保了无论逗号是否存在，都不会改变documents的语意，以避免了和其他语言因为逗号而报错。处于源代码的可读性和我可维护性的考虑，使用两个逗号作为行终止符也被允许。


### 词汇tokens (Lexical Tokens)

Token ::
  - 标点(Punctuator)
  - 名称(Name)
  - int数值(IntValue)
  - 浮点数值(FloatValue)
  - 字符串值(StringValue)

GraphQL文档由几种不可分割的词汇token组成，通过源Unicode字符的特定模式在词法语法中定义。


token在GraphQL查询文档句法中被用作终端符号语法。


### 被跳过的token(Ignored Tokens)

跳过 ::
  - BOM符号(UnicodeBOM)
  - 空白(WhiteSpace)
  - 换行(LineTerminator)
  - 注释(Comment)
  - 逗号(Comma)

每个词汇token之前和之后可能是任意数量的被忽略的token包括{WhiteSpace}和{Comment}。 没有忽略的token重要的，不过某些一般被忽略的字符可能会出现在一个有意义的词法标记中，例如{String}可能包含白色空格字符。

在解析给定的token时，不会忽略任何字符，没有例外。比如定义一个{floatValue}时候，字符之间允许使用空白字符。


### 标点(Punctuator)

标点(Punctuator) ::  

- `! $ ( ) ... : = @ [ ] { | }`

GraphQL文档包括了标点符号以便描述结构。GraphQL是一种数据描述语言，而不是一种编程语言，因此GraphQL缺少经常用来描述数学表达式的标点符号。


### 名称(Name)

名称(Name) :: 

- `/[_A-Za-z][_0-9A-Za-z]*/`

GraphQL文档中有数种名称(Name): 操作(operations), 域(fields), 参数(arguments),
指令(directives), 片段(fragments), 变量(variables). 所有的名称(Name)必须遵循相同的语法形式.

名称(Name)对大消息敏感，`name`, `NAME`是两个不同的名称(Name)。 下划线也是不可忽视的，`other_name`和`othername`是两个不同的名字。

GraphQL中的名称限于此可能的<acronym>ASCII</acronym>子集，以便能良好和其他系统交互。


## 查询文档(Query Document)

文档(Document)定义

定义 :
  - 操作定义／OperationDefinition
  - 片段定义／FragmentDefinition

GraphQL查询文档描述了一个可以被GraphQL服务接受的完整的文件或请求字符串。 文档包含操作的多个定义片段（FragmentDefinition）。 当query document包含了任意操作的时候，可以被服务器解析，但是不包含操作的时候，也允许被解析和验证，以便允许客户端表示的单个请求中包含了多个文件。

如果一个文档只包含一个操作，则该操作可能未命名或以缩写形式表示，省略了查询关键字和操作名称。 否则，如果GraphQL查询文档包含多个操作，每个操作都必须命名。 当提交查询文档时包含了对GraphQL服务的多个操作，所需操作的名称还必须提供才能执行。


## 操作(operations)

操作定义／OperationDefinition :
  - 操作类型名称？ 变量定义， 指令， 选择集 ／OperationType Name? VariableDefinitions? Directives? SelectionSet
  - 选择集 ／ SelectionSet

操作类型 ／OperationType :  `query（查询）` `mutation（变更）` `subscription（订阅）`  中的一个

GraphQL模型有三种操作类型:

  * query（查询） - 一个只读请求.
  * mutation（变更） - 一个写入请求
  * subscription（订阅） - 一个长链接，以订阅响应源的特定数据

每个操作由可选的操作名称和选择集表示。

例如，这个变更操作可能会“喜欢”一个故事，然后检索新的喜欢数量：

```
mutation {
  likeStory(storyID: 12345) {
    story {
      likeCount
    }
  }
}
```

**查询简写**

如果文档仅包含一个查询操作，并且该查询定义了否变量并且不包含指令，该操作可以表述为简写形式，省略查询关键字和查询名称。

例如，这个未命名的查询操作是通过查询简写来编写的。

```graphql
{
  field
}
```

注意：下面很多例子将使用查询简写语法。

## 选择集(Selection Sets)

SelectionSet : { Selection+ }

（Selection） :
  - 域／ield
  - 片段解构／FragmentSpread
  - 行内片段／InlineFragment

一个操作选择了其需要的信息，并且会确定的收到查询的信息不会有额外的信息。有效避免了传统单次请求中数据过多或过少的情况。

```graphql
{
  id
  firstName
  lastName
}
```

在这个query中, `id`, `firstName`, 和 `lastName` 这些field处于同一个选择集(Selection Sets). 选择集(Selection Sets)也可以包含片段的定义.


## 域(Fields)

域(Fields) : Alias, Name Arguments, Directives, SelectionSet,

选择集主要由字段组成。 一个域描述一个离散的可在选择集中请求的信息。

一些字段描述与其他数据的复杂数据或关系。为了进一步请求这个数据，一个字段本身可以包含一个选择集，允许深入嵌套的请求。所有GraphQL操作必须指定其选择集直到返回标量值的字段，以确保明确得到预定的响应。

例如，下面的操作表示了对于复杂数据的请求，其中的包含了更低层次的域，并描述了对应的字段结构。

```graphql
{
  me {
    id
    firstName
    lastName
    birthday {
      month
      day
    }
    friends {
      name
    }
  }
}
```

Fields in the top-level selection set of an operation often represent some
information that is globally accessible to your application and its current
viewer. Some typical examples of these top fields include references to a
current logged-in viewer, or accessing certain types of data referenced by a
unique identifier.

```graphql
# `me` could represent the currently logged in viewer.
{
  me {
    name
  }
}

# `user` represents one of many users in a graph of data, referred to by a
# unique identifier.
{
  user(id: 4) {
    name
  }
}
```


## 参数(Arguments)

Arguments : ( Argument+ )

Argument : Name : Value

域(Fields)是一个概念上的函数，有时候也会接受特定的参数. 这些参数通常直接映射到GraphQL服务器中对应的函数实现之中。

在下面的例子中，我们要查询一个特定的用户(通过`id`这个参数来请求)并且将图片限制在一个特定大小上，通过`size`这个参数:

```graphql
{
  user(id: 4) {
    id
    name
    profilePic(size: 100)
  }
}
```

多参数可以在同一个域中共存:

```graphql
{
  user(id: 4) {
    id
    name
    profilePic(width: 100, height: 50)
  }
}
```

**参数(Arguments) 是无顺序的！**

参数可以用任意的句法顺序表示，并拥有同等的语意。

以下两个query是等效的。

```graphql
{
  picture(width: 200, height: 100)
}
```

```graphql
{
  picture(height: 100, width: 200)
}
```


## 域别称(Field Alias)

Alias : Name :

默认情况下，响应得到的对象中的key使用字段名词，但是你可以使用制定别名来定义不同的名称。

在这个例子中，我们可以使用别称来获得两个不同大小的图片，并确保生成的对象中不会有重复的key。

```graphql
{
  user(id: 4) {
    id
    name
    smallPic: profilePic(size: 64)
    bigPic: profilePic(size: 1024)
  }
}
```

将会返回以下的结果:

```js
{
  "user": {
    "id": 4,
    "name": "Mark Zuckerberg",
    "smallPic": "https://cdn.site.io/pic-4-64.jpg",
    "bigPic": "https://cdn.site.io/pic-4-1024.jpg"
  }
}
```

查询集的顶层域依然是一个字段，所以也可以设置别称

```graphql
{
  zuck: user(id: 4) {
    id
    name
  }
}
```

将会返回以下的结果:

```js
{
  "zuck": {
    "id": 4,
    "name": "Mark Zuckerberg"
  }
}
```

如果提供了别称，则字段的响应key是其别称，否则为该字段的名称。


## 片段 ／ Fragments

片段解构／FragmentSpread : ... FragmentName Directives?

片段定义／FragmentDefinition : fragment FragmentName TypeCondition Directives? SelectionSet

片段名称／FragmentName : 除了`on`之外的名称

片段是GraphQL中组成的主要单位。

片段允许重用常用的重复选择的字段，减少
文档中重复的文本。 内联片段可以直接在一个
在对接口进行查询时根据类型条件进行选择
或联合。

例如，如果我们想获取一些关于共同的朋友的常见信息
还有一些用户的朋友：

```graphql
query noFragments {
  user(id: 4) {
    friends(first: 10) {
      id
      name
      profilePic(size: 50)
    }
    mutualFriends(first: 10) {
      id
      name
      profilePic(size: 50)
    }
  }
}
```

可以将重复的字段提取成片段并由其组成
父片段或查询。

```graphql
query withFragments {
  user(id: 4) {
    friends(first: 10) {
      ...friendFields
    }
    mutualFriends(first: 10) {
      ...friendFields
    }
  }
}

fragment friendFields on User {
  id
  name
  profilePic(size: 50)
}
```

使用解构运算符(`...`)使用片段.  All fields selected
by the fragment will be added to the query field selection at the same level
as the fragment invocation. This happens through multiple levels of fragment
spreads.

所有被片段选中的域会作为片段文字添加到同一个层级的查询域选择集合之中。这个事情也会发生在多层的片段解构。

举个例子:

```graphql
query withNestedFragments {
  user(id: 4) {
    friends(first: 10) {
      ...friendFields
    }
    mutualFriends(first: 10) {
      ...friendFields
    }
  }
}

fragment friendFields on User {
  id
  name
  ...standardProfilePic
}

fragment standardProfilePic on User {
  profilePic(size: 50)
}
```

所有无片段的，有片段的，嵌套片段的的查询会产生同样的响应。


### 类型条件／Type Conditions

TypeCondition : on NamedType

片段必须指定它们所适用的类型. 这个例子中, `friendFields`可以用在查询`User`的上下文之中.

不能在任何输入值（标量，枚举或输入）上指定片段。

可以在对象类型，接口和联合体上指定片段。

片段中的选择只有当具体类型的对象操作符合片段的类型时才返回值。

例如在Facebook数据模型的这个查询中：

```graphql
query FragmentTyping {
  profiles(handles: ["zuck", "cocacola"]) {
    handle
    ...userFragment
    ...pageFragment
  }
}

fragment userFragment on User {
  friends {
    count
  }
}

fragment pageFragment on Page {
  likers {
    count
  }
}
```

`profiles`根字段返回一个列表，其中每个元素可以是“Page”或“User”。当“profiles”结果中的对象是“User”时，“friends”将会出现，“likers”不会。相反，当结果是一个“页面”，“喜好者”将会出现，“朋友”不会。

```js
{
  "profiles": [
    {
      "handle": "zuck",
      "friends": { "count" : 1234 }
    },
    {
      "handle": "cocacola",
      "likers": { "count" : 90234512 }
    }
  ]
}
```


### 行内片段／Inline Fragments

InlineFragment : ... TypeCondition? Directives? SelectionSet

片段可以在选择集内部内联定义。这样做有条件地包括基于其运行时类型的字段。在`query FragmentTyping`示例中演示了标准片段包含的特征。我们可以使用内联片段来完成同样的事情。

```graphql
query inlineFragmentTyping {
  profiles(handles: ["zuck", "cocacola"]) {
    handle
    ... on User {
      friends {
        count
      }
    }
    ... on Page {
      likers {
        count
      }
    }
  }
}
```

也可以使用内联片段将指令应用于一组字段。如果省略了TypeCondition，则内联片段被认为与封闭上下文具有相同的类型。

```graphql
query inlineFragmentNoType($expandedInfo: Boolean) {
  user(handle: "zuck") {
    id
    name
    ... @include(if: $expandedInfo) {
      firstName
      lastName
      birthday
    }
  }
}
```


## Input Values

Value[Const] :
  - [~Const] Variable
  - IntValue
  - FloatValue
  - StringValue
  - BooleanValue
  - NullValue
  - EnumValue
  - ListValue[?Const]
  - ObjectValue[?Const]

Field and directive arguments accept input values of various literal primitives;
input values can be scalars, enumeration values, lists, or input objects.

If not defined as constant (for example, in {DefaultValue}), input values can be
specified as a variable. List and inputs objects may also contain variables (unless defined to be constant).


### Int Value

IntValue :: IntegerPart

IntegerPart ::
  - NegativeSign? 0
  - NegativeSign? NonZeroDigit Digit*

NegativeSign :: -

Digit :: one of 0 1 2 3 4 5 6 7 8 9

NonZeroDigit :: Digit but not `0`

An Int number is specified without a decimal point or exponent (ex. `1`).


### Float Value

FloatValue ::
  - IntegerPart FractionalPart
  - IntegerPart ExponentPart
  - IntegerPart FractionalPart ExponentPart

FractionalPart :: . Digit+

ExponentPart :: ExponentIndicator Sign? Digit+

ExponentIndicator :: one of `e` `E`

Sign :: one of + -

A Float number includes either a decimal point (ex. `1.0`) or an exponent
(ex. `1e50`) or both (ex. `6.0221413e23`).


### Boolean Value

BooleanValue : one of `true` `false`

The two keywords `true` and `false` represent the two boolean values.


### String Value

StringValue ::
  - `""`
  - `"` StringCharacter+ `"`

StringCharacter ::
  - SourceCharacter but not `"` or \ or LineTerminator
  - \u EscapedUnicode
  - \ EscapedCharacter

EscapedUnicode :: /[0-9A-Fa-f]{4}/

EscapedCharacter :: one of `"` \ `/` b f n r t

Strings are sequences of characters wrapped in double-quotes (`"`). (ex.
`"Hello World"`). White space and other otherwise-ignored characters are
significant within a string value.

Note: Unicode characters are allowed within String value literals, however
GraphQL source must not contain some ASCII control characters so escape
sequences must be used to represent these characters.

**Semantics**

StringValue :: `""`

  * Return an empty Unicode character sequence.

StringValue :: `"` StringCharacter+ `"`

  * Return the Unicode character sequence of all {StringCharacter}
    Unicode character values.

StringCharacter :: SourceCharacter but not `"` or \ or LineTerminator

  * Return the character value of {SourceCharacter}.

StringCharacter :: \u EscapedUnicode

  * Return the character whose code unit value in the Unicode Basic Multilingual
    Plane is the 16-bit hexadecimal value {EscapedUnicode}.

StringCharacter :: \ EscapedCharacter

  * Return the character value of {EscapedCharacter} according to the table below.

| Escaped Character | Code Unit Value | Character Name               |
| ----------------- | --------------- | ---------------------------- |
| `"`               | U+0022          | double quote                 |
| `\`               | U+005C          | reverse solidus (back slash) |
| `/`               | U+002F          | solidus (forward slash)      |
| `b`               | U+0008          | backspace                    |
| `f`               | U+000C          | form feed                    |
| `n`               | U+000A          | line feed (new line)         |
| `r`               | U+000D          | carriage return              |
| `t`               | U+0009          | horizontal tab               |


### Null Value

NullValue : `null`

Null values are represented as the keyword {null}.

GraphQL has two semantically different ways to represent the lack of a value:

  * Explicitly providing the literal value: {null}.
  * Implicitly not providing a value at all.

For example, these two field calls are similar, but are not identical:

```graphql
{
  field(arg: null)
  field
}
```

The first has explictly provided {null} to the argument "arg", while the second
has implicitly not provided a value to the argument "arg". These two forms may
be interpreted differently. For example, a mutation representing deleting a
field vs not altering a field, respectively. Neither form may be used for an
input expecting a Non-Null type.

Note: The same two methods of representing the lack of a value are possible via
variables by either providing the a variable value as {null} and not providing
a variable value at all.


### Enum Value

EnumValue : Name but not `true`, `false` or `null`

Enum values are represented as unquoted names (ex. `MOBILE_WEB`). It is
recommended that Enum values be "all caps". Enum values are only used in
contexts where the precise enumeration type is known. Therefore it's not
necessary to supply an enumeration type name in the literal.


### List Value

ListValue[Const] :
  - [ ]
  - [ Value[?Const]+ ]

Lists are ordered sequences of values wrapped in square-brackets `[ ]`. The
values of a List literal may be any value literal or variable (ex. `[1, 2, 3]`).

Commas are optional throughout GraphQL so trailing commas are allowed and repeated
commas do not represent missing values.

**Semantics**

ListValue : [ ]

  * Return a new empty list value.

ListValue : [ Value+ ]

  * Let {inputList} be a new empty list value.
  * For each {Value+}
    * Let {value} be the result of evaluating {Value}.
    * Append {value} to {inputList}.
  * Return {inputList}


### Input Object Values

ObjectValue[Const] :
  - { }
  - { ObjectField[?Const]+ }

ObjectField[Const] : Name : Value[?Const]

Input object literal values are unordered lists of keyed input values wrapped in
curly-braces `{ }`.  The values of an object literal may be any input value
literal or variable (ex. `{ name: "Hello world", score: 1.0 }`). We refer to
literal representation of input objects as "object literals."

**Input object fields are unordered**

Input object fields may be provided in any syntactic order and maintain
identical semantic meaning.

These two queries are semantically identical:

```graphql
{
  nearestThing(location: { lon: 12.43, lat: -53.211 })
}
```

```graphql
{
  nearestThing(location: { lat: -53.211, lon: 12.43 })
}
```

**Semantics**

ObjectValue : { }

  * Return a new input object value with no fields.

ObjectValue : { ObjectField+ }

  * Let {inputObject} be a new input object value with no fields.
  * For each {field} in {ObjectField+}
    * Let {name} be {Name} in {field}.
    * Let {value} be the result of evaluating {Value} in {field}.
    * Add a field to {inputObject} of name {name} containing value {value}.
  * Return {inputObject}


## Variables

Variable : $ Name

VariableDefinitions : ( VariableDefinition+ )

VariableDefinition : Variable : Type DefaultValue?

DefaultValue : = Value[Const]

A GraphQL query can be parameterized with variables, maximizing query reuse,
and avoiding costly string building in clients at runtime.

If not defined as constant (for example, in {DefaultValue}), a {Variable} can be
supplied for an input value.

Variables must be defined at the top of an operation and are in scope
throughout the execution of that operation.

In this example, we want to fetch a profile picture size based on the size
of a particular device:

```graphql
query getZuckProfile($devicePicSize: Int) {
  user(id: 4) {
    id
    name
    profilePic(size: $devicePicSize)
  }
}
```

Values for those variables are provided to a GraphQL service along with a
request so they may be substituted during execution. If providing JSON for the
variables' values, we could run this query and request profilePic of
size `60` width:

```js
{
  "devicePicSize": 60
}
```

**Variable use within Fragments**

Query variables can be used within fragments. Query variables have global scope
with a given operation, so a variable used within a fragment must be declared
in any top-level operation that transitively consumes that fragment. If
a variable is referenced in a fragment and is included by an operation that does
not define that variable, the operation cannot be executed.


## Input Types

Type :
  - NamedType
  - ListType
  - NonNullType

NamedType : Name

ListType : [ Type ]

NonNullType :
  - NamedType !
  - ListType !

GraphQL describes the types of data expected by query variables. Input types
may be lists of another input type, or a non-null variant of any other
input type.

**Semantics**

Type : Name

  * Let {name} be the string value of {Name}
  * Let {type} be the type defined in the Schema named {name}
  * {type} must not be {null}
  * Return {type}

Type : [ Type ]

  * Let {itemType} be the result of evaluating {Type}
  * Let {type} be a List type where {itemType} is the contained type.
  * Return {type}

Type : Type !

  * Let {nullableType} be the result of evaluating {Type}
  * Let {type} be a Non-Null type where {nullableType} is the contained type.
  * Return {type}


## Directives

Directives : Directive+

Directive : @ Name Arguments?

Directives provide a way to describe alternate runtime execution and type
validation behavior in a GraphQL document.

In some cases, you need to provide options to alter GraphQL's execution
behavior in ways field arguments will not suffice, such as conditionally
including or skipping a field. Directives provide this by describing additional information to the executor.

Directives have a name along with a list of arguments which may accept values
of any input type.

Directives can be used to describe additional information for types, fields, fragments
and operations.

As future versions of GraphQL adopt new configurable execution capabilities,
they may be exposed via directives.
