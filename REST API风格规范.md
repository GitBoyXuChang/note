## REST API风格规范

- URI(Uniform Resource Identifiers) 统一资源标示符
- URL(Uniform Resource Locator) 统一资源定位符

URI的格式定义如下：

```java
URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]
```

URL是URI的一个子集(一种具体实现)，对于REST API来说一个资源一般对应一个唯一的URI(URL)。在URI的设计中，我们会遵循一些规则，使接口看起透明易读，方便使用者调用。

### 总结的规则

规则1：URI结尾不应包含（/）
规则2：正斜杠分隔符（/）必须用来指示层级关系
规则3：应使用连字符（ - ）来提高URI的可读性
规则4：不得在URI中使用下划线（_）
规则5：URI路径中全都使用小写字母



### **1、关于分隔符“/”的使用**

对于REST API来说，"/"只是一个分隔符，并无其他含义。为了避免混淆，"/"不应该出现在URL的末尾。例如以下两个地址实际表示的都是同一个资源：

REST API对URI资源的定义具有唯一性，一个资源对应一个唯一的地址。为了使接口保持清晰干净，如果访问到末尾包含 "/" 的地址，服务端应该301到没有 "/"的地址上。当然这个规则也仅限于REST API接口的访问，对于传统的WEB页面服务来说，并不一定适用这个规则。

### **2、URI中尽量使用连字符"-"代替下划线"_"的使用**

使用下划线"*"来分割字符串(单词)可能会和链接的样式冲突重叠，而影响阅读性。但实际上，"-"和"*"对URL中字符串的分割语意上还是有些差异的："-"分割的字符串(单词)一般各自都具有独立的含义，可参见上面的例子。而"_"一般用于对一个整体含义的字符串做了层级的分割，方便阅读，例如你想在URL中体现一个ip地址的信息：210_110_25_88 .

### **3、URI中统一使用小写字母**

根据RFC3986定义，URI是对大小写敏感的，所以为了避免歧义，我们尽量用小写字符。但主机名(Host)和scheme（协议名称:http/ftp/...）对大小写是不敏感的。

### **4、URI中不要包含文件(脚本)的扩展名**

例如 .php .json 之内的就不要出现了，对于接口来说没有任何实际的意义。如果是想对返回的数据内容格式标示的话，通过HTTP Header中的Content-Type字段更好一些。

### 资源的原型

- **文档(Document)**

文档是资源的单一表现形式，可以理解为一个对象，或者数据库中的一条记录。在请求文档时，要么返回文档对应的数据，要么会返回一个指向另外一个资源(文档)的链接。以下是几个基于文档定义的URI例子：
`http://api.soccer.restapi.org/leagues/seattle` `http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet` `http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet/players/mike`

- **集合(Collection)**

集合可以理解为是资源的一个容器(目录)，我们可以向里面添加资源(文档)。例如：
`http://api.soccer.restapi.org/leagues` `http://api.soccer.restapi.org/leagues/seattle/teams` `http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet/players`

- **仓库(Store)**

仓库是客户端来管理的一个资源库，客户端可以向仓库中新增资源或者删除资源。客户端也可以批量获取到某个仓库下的所有资源。仓库中的资源对外的访问不会提供单独URI的，客户端在创建资源时候的URI除外。例如：
`PUT /users/1234/favorites/alonso`
上面的例子我们可以理解为，我们向一个id是1234的用户的仓库(收藏夹)中，添加了一个名为alonso的资源。通俗点儿说：就是用户收藏了一个自己喜爱的球员阿隆索。

- **控制器(Controller)**

控制器资源模型，可以执行一个方法，支持参数输入，结果返回。 是为了除了标准操作:增删改查(CRUD)以外的一些逻辑操作。控制器(方法)一般定义子URI中末尾，并且不会有子资源(控制器)。例如我们向用户重发ID为245743的消息：
`POST /alerts/245743/resend`

### URI命名规范

- 文档(Document)类型的资源用**名词(短语)单数**命名
- 集合(Collection)类型的资源用**名词(短语)复数**命名
- 仓库(Store)类型的资源用**名词(短语)复数**命名
- 控制器(Controller)类型的资源用**动词(短语)**命名
- URI中有些字段可以是变量，在实际使用中可以按需替换

例如一个资源URI可以这样定义：
`http://api.soccer.restapi.org/leagues/{leagueId}/teams/{teamId}/players/{playerId}`
其中：leagueId,teamId,playerId 是变量(数字，字符串都类型都可以)。

- **CRUD**的操作不要体现在URI中，HTTP协议中的操作符已经对CRUD做了映射。

CRUD是创建，读取，更新，删除这四个经典操作的简称
例如删除的操作用REST规范执行的话，应该是这个样子：
`DELETE /users/1234`

以下是几个错误的示例：
`GET /deleteUser?id=1234`
`GET /deleteUser/1234`
`DELETE /deleteUser/1234`
`POST /users/1234/delete`

### URI的query字段

```
http://api.college.restapi.org/students/morgan/send-sms`
`http://api.college.restapi.org/students/morgan/send-sms?text=hello
```

以上的两个URI看起来很像，但实际的含义是有差别的。第一个URI是一个发送消息的Controller类型的API，第二个URI是发送一个text的内容是hello的消息。

在REST中,query字段一般作为查询的参数补充，也可以帮助标示一个唯一的资源。但需要注意的是，作为一个提供查询功能的URI，无论是否有query条件，我们都应该保证结果的唯一性，一个URI对应的返回数据是不应该被改变的(在资源没有修改的情况下)。HTTP中的缓存也可能缓存查询结果，这个也是我们需要知道的。

- Query参数可以作为Collection或Store类型资源的过滤条件来使用

例如：
`GET /users //返回所有用户列表`
`GET /users?role=admin //返回权限为admin的用户列表`

- Query参数可以作为Collection或Store资源列表分页标示使用

如果是一个简单的列表操作，可以这样设计：
`GET /users?pageSize=25&pageStartIndex=50`
如果是一个复杂的列表或查询操作的话，我们可以为资源设计一个Collection，因为复杂查询可能会涉及比较多的参数，建议使用Post的方式传入，例如这样：
`POST /users/search`

## HTTP交互设计

### HTTP请求方法的使用

- **GET**方法用来获取资源
- **PUT**方法可用来新增/更新Store类型的资源
- **PUT**方法可用来更新一个资源
- **POST**方法可用来创建一个资源
- **POST**方法可用来触发执行一个Controller类型资源
- **DELETE**方法用于删除资源

一旦资源被删除，GET/HEAD方法访问被删除的资源时，要返回404
DELETE是一个比较纯粹的方法，我们不能对其做任何的重构或者定义，不可附加其它状态条件，如果我们希望"软"删除一个资源，则这种需求应该由Controller类资源来实现。

### HTTP响应状态码的使用

- **200 (“OK”)** 用于一般性的成功返回，不可用于请求错误返回
- **201 (“Created”)** 资源被创建
- **202 (“Accepted”)** 用于Controller控制类资源异步处理的返回，仅表示请求已经收到。对于耗时比较久的处理，一般用异步处理来完成
- **204 (“No Content”)** 此状态可能会出现在PUT、POST、DELETE的请求中，一般表示资源存在，但消息体中不会返回任何资源相关的状态或信息。
- **301 (“Moved Permanently”)** 资源的URI被转移，需要使用新的URI访问
- **302 (“Found”)** 不推荐使用，此代码在HTTP1.1协议中被303/307替代。我们目前对302的使用和最初HTTP1.0定义的语意是有出入的，应该只有在GET/HEAD方法下，客户端才能根据Location执行自动跳转，而我们目前的客户端基本上是不会判断原请求方法的，无条件的执行临时重定向
- **303 (“See Other”)** 返回一个资源地址URI的引用，但不强制要求客户端获取该地址的状态(访问该地址)
- **304 (“Not Modified”)** 有一些类似于204状态，服务器端的资源与客户端最近访问的资源版本一致，并无修改，不返回资源消息体。可以用来降低服务端的压力
- **307 (“Temporary Redirect”)** 目前URI不能提供当前请求的服务，临时性重定向到另外一个URI。在HTTP1.1中307是用来替代早期HTTP1.0中使用不当的302
- **400 (“Bad Request”)** 用于客户端一般性错误返回, 在其它4xx错误以外的错误，也可以使用400，具体错误信息可以放在body中
- **401 (“Unauthorized”)** 在访问一个需要验证的资源时，验证错误
- **403 (“Forbidden”)** 一般用于非验证性资源访问被禁止，例如对于某些客户端只开放部分API的访问权限，而另外一些API可能无法访问时，可以给予403状态
- **404 (“Not Found”)** 找不到URI对应的资源
- **405 (“Method Not Allowed”)** HTTP的方法不支持，例如某些只读资源，可能不支持POST/DELETE。但405的响应header中必须声明该URI所支持的方法
- **406 (“Not Acceptable”)** 客户端所请求的资源数据格式类型不被支持，例如客户端请求数据格式为application/xml，但服务器端只支持application/json
- **409 (“Conflict”)** 资源状态冲突，例如客户端尝试删除一个非空的Store资源
- **412 (“Precondition Failed”)** 用于有条件的操作不被满足时
- **415 (“Unsupported Media Type”)** 客户所支持的数据类型，服务端无法满足
- **500 (“Internal Server Error”)** 服务器端的接口错误，此错误于客户端无关

## 原数据设计

### HTTP Headers

- **Content-Type** 标示body的数据格式
- **Content-Length** body 数据体的大小，客户端可以根据此标示检验读取到的数据是否完整，也可以通过Header判断是否需要下载可能较大的数据体
- **Last-Modified** 用于服务器端的响应，是一个资源最后被修改的时间戳，客户端(缓存)可以根据此信息判断是否需要重新获取该资源
- **ETag** 服务器端资源版本的标示，客户端(缓存)可以根据此信息判断是否需要重新获取该资源，需要注意的是，ETag如果通过服务器随机生成，可能会存在多个主机对同一个资源产生不同ETag的问题
- Store类型的资源要支持有条件的PUT请求

假设有两个客户端client#1/#2都向一个Store资源提交PUT请求，服务端是无法清楚的判断是要insert还是要update的，所以我们要在header中加入条件标示if-Match，If-Unmodified-Since 来明确是本次调用API的意图。例如：

client#1第一次向服务端发起一个请求 PUT /objects/2113 此时2113资源还不存在，那服务端会认为本次请求是一个insert操作，完成后，会返回 201 (“Created”)

client#2再一次向服务端发起同一个请求 PUT /objects/2113 时，因2113资源已存在，服务端会返回 409 (“Conflict”)

为了能让client#2的请求成功，或者说我们要清楚的表明本次操作是一次update操作，我们必须在header中加入一些条件标示，例如 if-Match。我们需要给出资源的ETag(if-Match:Etag)，来表明我们希望更新资源的版本，如果服务端版本一致，会返回200 (“OK”) 或者 204 (“No Content”)。如果服务端发现指定的版本与当前资源版本不一致，会返回 412 (“Precondition Failed”)

- **Location** 在响应header中使用，一般为客户端感兴趣的资源URI,例如在成功创建一个资源后，我们可以把新的资源URI放在Location中，如果是一个异步创建资源的请求，接口在响应202 (“Accepted”)的同时可以给予客户端一个异步状态查询的地址
- **Cache-Control, Expires, Date** 通过缓存机制提升接口响应性能,同时根据实际需要也可以禁止客户端对接口请求做缓存。对于REST接口来说，如果某些接口实时性要求不高的情况下，我们可以使用**max-age**来指定一个小的缓存时间，这样对客户端和服务器端双方都是有利的。一般来说只对GET方法且返回200的情况下使用缓存，在某些情况下我们也可以对返回3xx或者4xx的情况下做缓存，可以防范错误访问带来的负载。
- 我们可以自定义一些头信息，作为客户端和服务器间的通信使用，但不能改变HTTP方法的性质。自定义头尽量简单明了，不要用body中的信息对其作补充说明。

### 数据媒体类型(Media Type)

定义如下：

```html
Content-Type: type "/" subtype *( ";" parameter )
两个实例：
Content-type: text/html; charset=ISO-8859-4
Content-type: text/plain; charset="us-ascii"
```

type 主类型一般为：application, audio, image, message, model, multipart, text, video。REST接口的主类型一般使用**application**

### 数据媒体类型(Media Type)设计

- 设计上来说，服务器端可以支持多种媒体类型
- 可以通过URI的查询字段来指定客户端希望的数据类型

```html
GET /bookmarks/mikemassedotcom?accept=application/xml
```

## 数据媒体格式的设计

### body的媒体格式

- json是一种流行且轻便友好的格式，json是一种无序的键值对的集合，其中key是需要用双引号引起来的，value如果是数字可以不用双引号，如果是非数字的格式需要使用双引号。

```html
这是一个json格式的例子：
{
"firstName" : "Osvaldo",
"lastName" : "Alonso", "firstNamePronunciation" : "ahs-VAHL-doe", "number" : 6,
"birthDate" : "1985-11-11"
}
```

- json是允许大小写混用命名的，但要避免使用特殊符号
- 除了json我们也可以使用其他常用的格式，例如xml,html等
- body本身只应包含资源相关的信息，不要附加其它传输状态的信息

### 错误响应描述

- 错误信息的格式应该保持一致，例如用以下方式(json格式):

```html
{
  "id" : Text,  //错误唯一标示id
  "description" : Text  //错误具体描述
}

如果有多个错误，可以用json数组来描述：
{
  "elements" : [
    {
      "id" : "Update Failed",
      "description" : "Failed to update /users/1234"
    }
  ]
}
```

- 错误类型需要保持统一

## 客户端关注的问题

### 接口版本管理

- 一个资源，只用一种单一的URI来标示，资源的版本不应该体现在URI中
- 资源的版本是可以由客户端来指定的，并且提供向后兼容
- ETag可以用来管理资源的版本，有助于客户端缓存的应用

### 接口的安全

- 使用OAuth认证，对敏感资源保护
- 使用API管理策略，或管理平台（Apigee, Mashery）

### 接口数据响应的结构

- 客户端可以指定接口返回需要的资源字段，或者指定不希望返回的字段，这样有助于提升接口交互的效率，较少带宽的浪费

```json
只获许部分字段：
GET /students/morgan?fields=(firstName, birthDate)

不希望获取某些字段：
GET /students/morgan?fields=!(address,schedule!(wednesday, friday))
```

- 资源数据中可以包含嵌入式链接，用来描述查询资源的子集，我们也可以传入相关参数，要求服务端替换链接为实际的数据

```json
{
  "firstName" : "Morgan", 
  "birthDate" : "1992-07-31",
  # Other fields...
  "links" : {
    "self" : {
      "href" : "http://api.college.restapi.org/students/morgan",
      "rel" : "http://api.relations.wrml.org/common/self" 
    },
    "favoriteClass" : {
      "href" : "http://api.college.restapi.org/classes/japn-301",    
      "rel" : "http://api.relations.wrml.org/college/favoriteClass"
    },
    # Other links... 
  }
}

如果我们传入embed=(favoriteClass)的参数，返回的数据中将用实际的内容替换links里的对应的潜入资源：
# Request
GET /students/morgan?embed=(favoriteClass)

# Response
{
  "firstName" : "Morgan",
  "birthDate" : "1992-07-31", 
  "favoriteClass" : { //需要返回的嵌入数据
    "id" : "japn-301",
    "name" : "Third-Year Japanese", 
    "links" : {
      "self" : {
        "href" : "http://api.college.restapi.org/classes/japn-301",     
        "rel" : "http://api.relations.wrml.org/common/self"
      } 
    }
}

# Other fields...
  "links" : { 
    "self" : {
      "href" : "http://api.college.restapi.org/students/morgan",
      "rel" : "http://api.relations.wrml.org/common/self" 
    },
    # 之前的嵌入式链接favoriteClass,已被替换为实体数据
    # Other links... 
  }
}


#其中嵌入式链接信息中的 rel ,一般是对 href 资源如何交互的描述，例如是通过 GET 还是 POST 方法，可以是以下的结构:
{
  "name": "morgan",
  "method": "GET", 
  ... #其它描述字段
}
```

### JavaScript客户端

目前主流的浏览器对JavaScript的支持越来越完善，因此对于WEB应用来说，我们完全可以把客户单看成一个JavaScript客户端。

- 一般浏览器对于跨域的操作都有一定的安全策略，通常我们可以使用JSONP来解决跨域接口访问的限制
- 通过CORS(Cross-Origin Resource Sharing)来解决跨域访问，此方法与JSONP相比，支持更多的方法，JSONP只能用于GET请求, 一般现代的浏览器会支持CORS的方式