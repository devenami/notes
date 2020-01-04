## Proto3参考文档

该引用文档描述了如何使用`protocol buffer`语言来构造你自己的`protocol buffer`数据，包含`.proto`文件语法和如何通过`.proto`生成可访问的数据。

### 定义一个消息类型

第一步，让我们来看一个非常简单的例子。现在你想定义一个搜索请求的消息格式，每个搜索请求有一个查询字符串、你感兴趣的当前结果的页码以及一个每页显示的结果数。在这里你使用了一个`.proto`文件定义了一个消息类型。

```protobuf
syntax = "proto3"

message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_page = 3;
}
```

* 在文件的第一行我们使用了`.proto3`语法：如果你没有定义这一行，那么`protocol buffer`编译器会认为当时使用的是`proto2`。这必须是文件的第一行非空，非注释行。
* `SearchRequest`的消息定义指定了三个属性(成对的 名称/值)，数据的每一片都包含了消息的类型。每个属性都有一个名字和类型。

#### 指定属性的类型

在上面的例子中，所有的属性都是`标量类型`：两个整形(`page_number`和`result_pre_page`)和一个字符串类型(`query`)，当然，你也可以为属性指定`复合类型`，包括`枚举`和其他的消息类型。

#### 指定属性数字

就像你看到的，在消息中定义的每个属性都有一个唯一的数字。这个属性数字用于在消息的二进制格式中区分属性，并且你不能改变已被使用的消息类型的数字。需要注意的是数字值在`1-15`之间的会使用一个字节(byte)进行编码，包含了属性数字和属性类型。属性数字在`16-2047`的会使用两个字节。你应该给经常出现的消息元素提供`1-15`之间的数字。切记为将来可能添加的频繁出现的元素留出一些空间。

最小的属性数字可以被指定为`1`，最大可以被指定为`2^29 - 1`或者`536870911`。你不能使用`19000-19999`（`FieldDescriptor::kFirstReservedNumber` - `FieldDescriptor::kLastReservedNumber`）之间的数字，它们都提供给了`protocol buffer`的实现。

#### 指定属性的规则

消息属性可以是下面的任意一种：

* 单一类型：众所周知的消息属性可以有0个或一个值(但是不会多于1个)。并且这是`proto3`语法默认的属性规则。
* **重复类型**：这种消息的属性可以被重复任意次(包含0次)，被重复元素的顺序会被完整的保存下来。

在`proto3`中，`标量数字类型`的字段默认情况相爱使用`packed`进行编码。

#### 添加更多的消息类型

多个消息类型可以定义在一个`.proto`文件中。如果你定义了多个相关的消息这是很有用的 - 就像下面的例子。如果你想定义一个回复消息对应了就是`SearchResponse`消息类型，你可以在相同的`.proto`文件中添加它：

```protobuf
message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_page = 3;
}

message SearchResponse {
	...
}
```

#### 添加注释

在`.proto`文件中添加注释使用的是`C/C++`风格的`//`和`/**/`语法。

```protobuf
/* SearchRequest represents a search quer, with pagination options to
 * indicate whilch results to include in the response. */
message SearchRequest {
	string query = 1;
	int32 page_number = 2;	// Which page number do we want?
	int32 result_per_page = 3;	// Number of results to return per page.
}
```

#### 保留属性

如果你通过完全移除字段或将其注释来更新消息类型，将来的用户可以在对类型进行更新时重用该字段号。如果相同`.proto`文件的老版本在后来被使用，并且该文件中存在数据混乱或者bug，那么就会导致严重的问题。用于解决这种问题的一种方法是，指定被删除的属性数字(或者名字，但是会导致JSON序列化问题)为`保留的`。如果未来有用户尝试使用那些属性标识，那么编译器会提示用户。

```protobuf
message Foo {
	reserved 2, 15, 9 to 11;
	reserved "foo", "bar";
}
```

在同一个`reserved`内不能同时混用数字和名称。

#### 如何从`.proto`文件进行生成

当你在`.proto`文件上运行`protocol buffer`编译器时，编译器会生成你选择的语言，你可以使用你再文件中描述的消息类型进行工作，包含了获取和设置属性值，序列化消息到输出流及从输入流反序列化消息。

* C++，编译器从每个`.proto`文件中描述的消息类型生成了一个`.h`和`.cc`文件
* Java，编译器为每个消息类型生成一个`.java`文件，同时生成了一个特殊的`Builder`类用于创建消息实例。
* Python：Python编译器会生成一个模块，该模块带有`.proto`中每种消息类型的静态描述符，然后与原类一起使用，以在运行时创建必要的Python数据访问类。
* Go，编译器为每个在文件中的消息类型生成了一个`.pb.go`的文件

### 标量值类型

一个标量消息属性可以有下面的任意一种类型，下面的表展示了`.proto`文件中的特殊类型以及自动生成类型的对应关系。

| .proto Type | Notes                                                        | C++ Type | Java Type  | Python Type[2] | Go Type | Ruby Type                      | C# Type    | PHP Type          | Dart Type |
| :---------- | :----------------------------------------------------------- | :------- | :--------- | :------------- | :------ | :----------------------------- | :--------- | :---------------- | :-------- |
| double      |                                                              | double   | double     | float          | float64 | Float                          | double     | float             | double    |
| float       |                                                              | float    | float      | float          | float32 | Float                          | float      | float             | double    |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| uint32      | Uses variable-length encoding.                               | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| uint64      | Uses variable-length encoding.                               | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228. | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256. | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sfixed32    | Always four bytes.                                           | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sfixed64    | Always eight bytes.                                          | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| bool        |                                                              | bool     | boolean    | bool           | bool    | TrueClass/FalseClass           | bool       | boolean           | bool      |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232. | string   | String     | str/unicode[4] | string  | String (UTF-8)                 | string     | string            | String    |
| bytes       | May contain any arbitrary sequence of bytes no longer than 232. | string   | ByteString | str            | []byte  | String (ASCII-8BIT)            | ByteString | string            |           |

### 默认值

当消息被解析时，如果被编码的消息没有包含指定的元素，那么那个属性被解析的对象将会被赋值默认值。下面是指定类型的默认值。

* string类型默认的是空字符串
* bytes类型默认的空字节
* bool类型默认为false
* 数值类型默认值为0
* 枚举类型默认值为第一个定义的类型，第一个属性的数字需要被设置为0
* 消息类型默认值不会被设置。它依赖于语言。

`repeated`属性的默认值为空(在适当的语言中通常是一个空列表)。

需要注意的是，对于一个标量类型，每个被解析的消息并没有办法知道该属性是否被明确的设置为默认值(例如一个boolean类型的值并无法知道是否被设置为false)或者没有被设置：当你定义消息类型是应非常小心。例如：如果你不想在默认的情况下boolean类型被设置为false时永远发生一种行为，那么你就不应该使用boolean的默认值。也要注意的是，如果标量的值被设置为默认值，那么该值也不会在写出的时候被序列化。

### 枚举

当你定义一个消息类型时，你可能只想要预定义的属性列表中的一个属性。你可以定义一个`enum`到你的消息中，并为每一个可能的值指定一个常量。

```protobuf
message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_page = 3;
	enum Corpus {
		UNIVERSAL = 0;
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
	}
	Corpus corpus = 4;
}
```

就像你看到的，`Corpus`的第一个属性被设置为0：每个枚举定义**必须**包含第一个元素且被映射为0。这是因为：

* 它被设置为0值，那么我们就可以使用0作为数值类型的默认值
* 0值必须被放在第一个元素是是因为兼容`proto2`的第一个属性值为枚举的默认值

你可以为相同的枚举值定义别名实现不同的枚举常量。为了实现这种效果你需要设置`allow_alias`的可选项为`true`，否则编译时就会出错。

```protobuf
enum EnumAllowingAlias {
	option allow_alias = true;
	UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
}
enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

枚举常量的值必须在32为整形的范围内。

### 使用其他的消息类型

你可以使用其他的消息类型作为属性类型。

```protobuf
message SearchResponse {
    repeated Result results = 1;
}

message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```

#### import 定义

在上面例子中，`Result`消息类型被定义在`SearchResponse`相同的文件中。

你可以使用定义在其他`.proto`文件的消息类型，并使用`import`导入他们。导入其他的`.proto`文件的定义，你需要在文件中添加一个导入块。

```protobuf
import "myproject/other_protos.proto";
```

默认情况下，你只能使用直接导入的`.proto`文件中的定义。但是有些情况下你需要将`.proto`文件移动到一个新的位置。现在你可以将虚拟的`.proto`文件放在旧位置，而不是直接移动`.proto`文件并一次更改所有被调用的位置，而是使用导入公共概念将所有导入转发到新位置。任何导入的`proto`包含`import public`块，它的依赖就可以向下进行传递。

```protobuf
// new.proto
// 所有的定义都被移动到这里
```

```protobuf
// old.proto
// 这个proto被所有的客户端导入
import public "new.proto"
import "other.proto"
```

```protobuf
// client.proto
import "old.proto"
// 你可以使用old.proto和new.proto中的定义项，但是不能使用other.proto的定义项
```

protocol编译器使用在编译命令行上使用`-I`或`--proto_path`标记设置的一系列目录搜索被导入的文件。如果没有设置标记，那么编译器会搜索编译器被调用的目录。常规情况下，你应该设置`--proto_path`标记作为项目的根目录，并为所有被导入的元素使用全限定名。

### 嵌套类型

你可以在其他消息类型中定义和使用消息。

```protobuf
message SearchResponse {
    message Result {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result results = 1;
}
```

如果你想在父消息体之外使用消息，那么你需要引用`Parent.Type`:

```protobuf
message SomeOtherMessage {
    SearchResponse.Result result = 1;
}
```

也可以更深层的嵌套消息结构：

```protobuf
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

### 更新消息类型

更新消息类型需要符合以下规则：

* 不用改变任何已存在属性的数字
* 如果你添加了新的属性，任何使用**旧**消息格式系列化的消息仍然可以被新生成的代码解析。你需要特别注意那么由于老代码产生的默认值。类似的，新代码创建的消息也可以被老代码解析：老代码会在解析时简单的忽略新的属性。
* 只要在你更新的消息类型中不会再次使用原来的数字，那么属性可以被移除。你可能项重命名属性或者添加一个前缀，或者使属性数字作为`reserved`(保留)，那么以后在你的`.proto`中不会重新使用该数字。
* `int32`,`uint32`,`int64`,`uint64`和`bool`都是兼容的。这意味着你可以在这些属性类型中向前或向后兼容。
* `sint32`和`sint64`互相兼容，但是和其他的整形类型是不兼容的。
* `string`和`bytes`互相兼容，也就是说它们都是验证过的`UTF-8`字节。
* 如果字节中包含一个消息的编码版本，那么嵌入的消息和`bytes`兼容。
* `fixed32`和`sfixed32`,`fixed64`，`sfixed64`兼容。
* `enum`和`int32`,`uint32`,`int64`,`uint64`在某些方面兼容（需要注意的是如果值不匹配则会被截断）。
* 改变一个单独的值到一个新的`oneof`是安全并且二进制兼容的。如果你能确保在同一时间只存在一个代码进行设值，那么移动多个属性到一个新的`oneof`可能是安全的。移动任意的属性到一个已存在的`oneof`是不安全的。

### 未知的属性

位置属性是格式正确的协议缓冲区序列化数据，表示解释器无法识别的属性。例如：当一个老的二进制解析器解析一个带有新属性的新的二进制数据时，那些新的属性在老的二进制中就是位置属性。

常规上，`proto3`消息在解析时始终会忽略位置的属性，但在3.5版本中，我们重新引入了保留位置属性以匹配`proto2`行为的功能。在3.5版本或更高的版本中，位置字段将在解析期间保留并包含在序列化输出中。

### Any

一个`Any`消息类型可以让你在不引入其他`.proto`定义的情况下作为一个嵌入式消息使用。`Any`包含任意序列化的消息(以字节为单位)以及URL，URL作为该消息的类型并解析为该消息的类型的全局唯一标识符。使用`Any`类型需要导入`google/protobuf/any.proto`

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

对于给定的消息类型，默认类型的URL是`type.googleapis.com/packagename.messagename`。

不同的语言实现将支持运行时库帮助程序以类型安全的方式打包或解压缩任何值。例如：在java中，Any类型会指定`pack()`和`unpack()`访问器，在C++中是`PackFrom()`和`UnpackTo()`方法。

```c++
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

### Oneof

如果你有一个很多属性的消息，并且所有的属性都会在同一时刻被设置，你可以使用`oneof`特性执行这种行为以节省内存。

类似于常规字段，共享内存字段除外，在同一时刻只能有一个属性被设值。设置`oneof`的任何成员会自动清除所有其他成员。你可以使用`case()`或`WhilchOneOf()`方法来检查`oneof`中的哪个值(如果有的话),具体取决于你选择的语言。

#### 使用 Oneof

在`.proto`中定义`oneof`，你应该使用`oneof`关键字并在后面紧跟`oneof`的名称：

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

现在添加了一个`oneof`属性到`oneof`的定义中，你可以添加任意类型的属性，但是并不能使用`repeated`类型的属性。

在生成的代码中，`oneof`属性拥有和常规属性一样的getter和setter方法。你也可以使用指定的方法来检查`oneof`的值是否被设置。

#### Oneof 特性

* 设置一个`oneof`的属性会自动清空其他`oneof`的成员。也就是说，如果你要设置多个`oneof`属性的值，只有最后一个你设置的属性才会有值。

  ```c++
  SampleMessage message;
  message.set_name("name");
  CHECK(message.has_name());
  message.mutable_sub_message();   // Will clear name field.
  CHECK(!message.has_name());
  ```

* 如果在解析时遇到相同的`oneof`的多个，只有最后一个成员会被用来解析消息。

* `oneof`内部的属性不能使用`repeated`。

* 反射API适用于`oneof`属性

* 如果你设置了`oneof`的属性为一个默认值，`oneof`属性的`case`会被设置，并且该值会在写出时序列化。

* 如果你使用C++，确保你的代码不会导致内存崩溃。下面的代码会崩溃，因为`sub+message`已经通过调用`set_name()`方法删除了。

  ```c++
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message
  sub_message->set_...  			// Crashes here
  ```

* 还是在C++中，如果你使用`Swap()`交换两个`oneof`消息，每个消息都会以对方的情况为准：下面的例子中，`msg1`会有一个`sub_message`，`msg2`会有一个`name`。

  ```c++
  SampleMessage msg1;
  msg1.set_name("name");
  SampleMessage msg2;
  msg2.mutable_sub_message();
  msg1.swap(&msg2);
  CHECK(msg1.has_sub_message());
  CHECK(msg2.has_name());
  ```

#### 向后兼容性问题

添加或移除一个`oneof`属性的术后要特别小心。如果检查`oneof`属性的值返回`None`/`NOT_SET`,这可能意味着`oneof`没有被设置或者在不同版本的`oneof`中设置了一个属性。由于无法知道导线上的未知字段是否是`oneof`的成员，因此无法分辨出差异。

### Maps

如果你想创建一个关联的map作为你数据定义的一部分，protocol buffers提供了一个简单的语法：

```protobuf
map<key_type, value_type> map_field = N;
```

这里的`key_type`可以是任意的整形获取字符串类型(除浮点类型和`bytes`类型外的任何标量)。枚举类型是一个非法的`key_type`。`value_type`可以是除了其他map类型的任意类型。

```protobuf
map<string, Project> projects = 3;
```

* map属性不能是`repeated`。
* 不能依赖map迭代的顺序。
* 当从一个`.proto`生成一个文本格式时，map会根据key进行排序。
* 当解析或者何明时，如果存在相同的key，最后的key会被使用。当从文本格式解析map时，如果遇到相同的key则会失败。
* 如果仅提供key并没有提供值，那么这种行为的序列化则会依赖语言。在`C++`,`Java`和`Python`中会使用默认值进行序列化，在其他语言中不会被序列化。

`proto3`支持的语言当前都已经支持了map的API。

#### 向后兼容性

map语法相当于下面的方式，所以不支持map的protocol实现仍然可以处理你的数据。

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

任何支持map的protocol实现必须同时可以生产和接受以上方式的定义。

### Packages

你可以添加一个可选的`package`说明符到服一个`.proto`文件中，以此来避免protocol消息类型之间的冲突。

```protobuf
package foo.bar;
message Open { ... }
```

当你定义消息类型的属性时，你可以使用package说明符：

```protobuf
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

这种包说明符的方式的影响范围依赖你选择的语言：

* 在`C++`中生成的类被包装在C++ namespace中。例如：`Open`将会在`foo::bar`命名空间中。
* 在`Java`中如果没有额外的指定`optional java_package`，那么package会被作为`Java package`使用。
* 在`Go`中，如果你没有使用`option go_package`，那么package会被用作go的包名。

### 定义 Service

如果你想在一个RPC(远程过程调用)系统中使用你的消息类型，你可以在一个`.proto`文件中定义一个 RPC 服务接口，protocol编译器会生成服务接口和所选语言的存根。例如，你想定义一个RPC方法，通过使用你的`SearchRequest`并返回一个`SearchResponse`，你可以向下面这样定义在`.proto`文件中：

```protobuf
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

### JSON 映射

`Proto3`支持JSON中的编码规范，使它更容易的在系统之间共享数据。下标按类型对编码进行了描述。

当把一个JSON编码数据解析为protocol buffer时，如果JSON数据中缺少数值或者它的值为`null`，那么该值会被解析为默认值。如果在protocol buffer中的属性有一个默认值，那么编码为JSON时，该值在默认情况下会被省略以节省空间。实现方可能提供是否忽略默认属性值的选项。

| proto3                 | JSON          | JSON example                             | Notes                                                        |
| :--------------------- | :------------ | :--------------------------------------- | :----------------------------------------------------------- |
| message                | object        | `{"fooBar": v, "g": null, …}`            | Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. If the `json_name` field option is specified, the specified value will be used as the key instead. Parsers accept both the lowerCamelCase name (or the one specified by the `json_name` option) and the original proto field name. `null` is an accepted value for all field types and treated as the default value of the corresponding field type. |
| enum                   | string        | `"FOO_BAR"`                              | The name of the enum value as specified in proto is used. Parsers accept both enum names and integer values. |
| map<K,V>               | object        | `{"k": v, …}`                            | All keys are converted to strings.                           |
| repeated V             | array         | `[v, …]`                                 | `null` is accepted as the empty list [].                     |
| bool                   | true, false   | `true, false`                            |                                                              |
| string                 | string        | `"Hello World!"`                         |                                                              |
| bytes                  | base64 string | `"YWJjMTIzIT8kKiYoKSctPUB+"`             | JSON value will be the data encoded as a string using standard base64 encoding with paddings. Either standard or URL-safe base64 encoding with/without paddings are accepted. |
| int32, fixed32, uint32 | number        | `1, -10, 0`                              | JSON value will be a decimal number. Either numbers or strings are accepted. |
| int64, fixed64, uint64 | string        | `"1", "-10"`                             | JSON value will be a decimal string. Either numbers or strings are accepted. |
| float, double          | number        | `1.1, -10.0, 0, "NaN","Infinity"`        | JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Exponent notation is also accepted. |
| Any                    | `object`      | `{"@type": "url", "f": v, … }`           | If the Any contains a value that has a special JSON mapping, it will be converted as follows: `{"@type": xxx, "value": yyy}`. Otherwise, the value will be converted into a JSON object, and the `"@type"` field will be inserted to indicate the actual data type. |
| Timestamp              | string        | `"1972-01-01T10:00:20.021Z"`             | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. |
| Duration               | string        | `"1.000340012s", "1s"`                   | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision, followed by the suffix "s". Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision and the suffix "s" is required. |
| Struct                 | `object`      | `{ … }`                                  | Any JSON object. See `struct.proto`.                         |
| Wrapper types          | various types | `2, "2", "foo", true, "true",null, 0, …` | Wrappers use the same representation in JSON as the wrapped primitive type, except that `null` is allowed and preserved during data conversion and transfer. |
| FieldMask              | string        | `"f.fooBar,h"`                           | See `field_mask.proto`.                                      |
| ListValue              | array         | `[foo, bar, …]`                          |                                                              |
| Value                  | value         |                                          | Any JSON value                                               |
| NullValue              | null          |                                          | JSON null                                                    |
| Empty                  | object        | {}                                       | An empty JSON object                                         |

#### JSON 选项

`Proto3` JSON实现方可能提供下面的选项：

* 忽略默认值的属性：`Proto3`默认会在输出时忽略默认值的属性。实现方可以提供该选项来覆盖这种行为，从而输出默认值属性。
* 忽略未知的属性：`Proto3` JSON 解析器默认会拒绝未知的属性，但是可以通过提供该选项来忽略在解析时忽略未知属性。
* 使用proto属性名替代首字母小写的驼峰命名法：默认情况下`proto3` JSON 打印器会转换属性名为小写的驼峰命名，并用户JSON字段名。实现者可以提供该选项使用proto属性名作为JSON的字段名。`Proto3` JSON 解析器需要同时转换小写的驼峰命名和属性名。
* 将枚举值作为正数而不是字符串发送：枚举值的名称默认会用作JSON的输出。可以提供该选项使用数字值替代枚举值。

### Options

在`.proto`文件中的个别声明可以通过一些`options`进行注解，Options并不会改变声明的完整意义，但是会在当前的上下文中影响处理的方式。完成的被支持的选项定义在`google/protobuf/descriptor.proto`中。

一些选项是文件级别的选项，这意味着他们会被声明在范围的顶级，而不是在任意的`message`,`enum`或`service`定义中。有些选项是消息级别的选项，意味值它们应该被定义在消息的定义中...

这里有一些比较常用的选项：

* `java_package`(文件选项)：你想在生成的java类中使用的包。如果没有生命该选项，默认使用`package`关键字作为类文件的包名。但是proto的`package`对java的包并不友好，因为proto的`package`并没有期望的域名开始。如果不是生成java代码，该选项不会有影响。

  ```protobuf
  option java_package = "com.example.foo";
  ```

* `java_multiple_files`(文件选项)：使顶级消息，枚举和服务在程序包级别定义，而不是在意`.proto`文件命名的外部类内部定义。

  ```protobuf
  option java_multiple_files = true;
  ```

* `java_outer_classname`(文件选项)：你想生成的最外层java类的名称（也是文件名称）。如果没有指定该选项，默认会改`.proto`文件名转换为驼峰格式。如果不是生成java代码，该选项不会有影响。

  ```protobuf
  option java_outer_classname = "Ponycopter";
  ```

* `optimize_for`(文件选项)：可以设置为`SPEED`,`CODE_SIZE`,`LITE_RUNTIME`，这会在一下的几种方式影响C++和java的代码生成。

  * `SPEED`(默认)：protocol buffer编译器会在消息类型上生成序列化、解析以及其他公共操作的代码。这种代码时最优化的。
  * `CODE_SIZE`：编译器会生成最小化的类文件并依赖共享，基于反射的代码实现序列化、解析以及不同的其他操作。特点是尺寸小，但是执行速度较慢。生成的API和`SPEED`一致。适用于存在大量`proto`文件并对性能没有要求的场景。
  * `LITE_RUNTIME`：编译器生成的类依赖于`lite`运行时库(`libprotobuf-lite`替代`libprotobuf`)。lite运行时比全类库的更小，但是会省略某些特性，如描述和反射。生成的类仅实现`MessageLite`接口，也进提供了少量的`Message`接口的方法。运行速度和`SPEED`模式一致，适用于移动端。

  ```protobuf
  option optimize_for = CODE_SIZE;
  ```

* `deprecated`(属性选项)：如果被设置为true，声明该属性已被废弃，并不应该在应该代码中使用。

  ```protobuf
  int32 old_field = 6 [deprecated=true];
  ```

### 生成类

生成Java Python C++ GO代码你需要定义一个`.proto`文件，然后在`.proto`文件上运行编译器`protoc`。

```shell
protoc  --proto_path=IMPORT_PATH \
		--cpp_out=DST_DIR \
        --java_out=DST_DIR \
        --python_out=DST_DIR \
        --go_out=DST_DIR \
        --ruby_out=DST_DIR \
        --objc_out=DST_DIR \
        --csharp_out=DST_DIR \
        path/to/file.proto
```

* `IMPORT_PATH`指定解析`import`指令时在其中查找`.proto`文件的目录。如果忽略，当前目录被会使用。多个`import`指令可以通过指定多个`--proto_path`选项，它们会顺序搜索。`-I`是`--proto_path`的简写形式。

* 你可以指定一个或多个输出指令：

  * `--cpp_out`在`DST_DIR`生成C++代码。
  * `--java_out`在`DST_DIR`生成Java代码。
  * `--python_out`在`DST_DIR`生成Python代码。
  * `--cgo_out`在`DST_DIR`生成GO代码。
  * ...

  如果`DST_DIR`以`.zip`或`.jar`结尾，编译器会输出到一个单独的使用给定名称的ZIP归档文档格式。`.jar`输出也会给出一个符合Java JAR规范的`mainfest`文件。需要注意的是，如果输出的归档文件已存在，那么会被覆盖；编译器没有聪明到向归档文件中添加文件。

* 你可以提供一个或多个`.proto`文件作为输入。可以一次指定多个`.proto`文件。尽管这些文件时相对于当前目录命名的，但是每个文件都必须位于`IMPORT_PATH`中，以便编译器可以确定其规范名称。

