# validator——v10

## 前言

**下载和导入**
```
go get github.com/go-playground/validator/v10

import "github.com/go-playground/validator/v10"
```

**参考文档**

```
https://godoc.org/github.com/go-playground/validator

https://github.com/go-playground/validator
```

***

## 特点

* 通过使用验证标签或自定义验证程序进行跨字段和跨结构验证

* 可以验证多维字段的所有级别

* 能够深入了解映射键和值进行验证

* 通过在验证之前确定其基础类型来处理类型接口

* 处理自定义字段类型，例如sql friver Valuer

* 别名验证标签，允许将多个验证映射到单个标签，以便更轻松地定义结构上的验证

* 提取自定义的字段名称，例如 可以指定在验证时提取JSON名称，并将其在结果FieldError中可用

* 可自定义的i18n感知错误消息。

***

## 总览

|  |  |  |
| :-----| :----- | :----- |
| Validation Functions Return Type error | Custom Validation Functions | Cross-Field Validation |
| Multiple Validators | Using Validator Tags | Baked In Validators and Tags |
| Skip Field | Or Operator | StructOnly |
| NoStructLevel | Omit Empty | Dive |
| Required | Required With | Required With All |
| Required Without | Required Without All | Is Default |
| Length | Maximum | Minimum |
| Equals | Not Equal | One Of |
| Greater Than | Greater Than or Equal | Less Than |
| Less Than or Equal | Field Equals Another Field | Field Does Not Equal Another Field |
| Field Greater Than Another Field | Field Greater Than Another Relative Field |Field Greater Than or Equal To Another Field |
| Field Greater Than or Equal To Another Relative Field | Less Than Another Field | Less Than Another Relative Field |
| Less Than or Equal To Another Field | Less Than or Equal To Another Relative Field | Field Contains Another Field |
| Field Excludes Another Field | Unique | Alpha Only |
| Alphanumeric | Alpha Unicode | Alphanumeric Unicode |
| Numeric | Hexadecimal String | Hexcolor String |
| RGB String | RGBA String | HSL String |
| HSLA String | E-mail String | File path |
| URL String | URI String | Urn RFC 2141 String |
| Base64 String | Base64URL String | Bitcoin Address|
| Ethereum Address | Contains | Contains Any |
|Contains Rune | Excludes | Excludes All |
|Excludes Rune | Starts With | Ends With |
| International Standard Book Number | International Standard Book Number 10 |International Standard Book Number 13 |
| Universally Unique Identifier UUID |Universally Unique Identifier UUID v3 |Universally Unique Identifier UUID v4 |
| Universally Unique Identifier UUID v5 | ASCII | Printable ASCII |
|Multi-Byte Characters |Data URL |Latitude |
|Longitude |Social Security Number SSN |Internet Protocol Address IP |
|Internet Protocol Address IPv4 |Internet Protocol Address IPv6 |Classless Inter-Domain Routing CIDR |
|Classless Inter-Domain Routing CIDRv4 |Classless Inter-Domain Routing CIDRv6 |Transmission Control Protocol Address TCP |
|Transmission Control Protocol Address TCPv4 |Transmission Control Protocol Address TCPv6 |User Datagram Protocol Address UDP |
|User Datagram Protocol Address UDPv4 |User Datagram Protocol Address UDPv6 |Internet Protocol Address IP |
|Internet Protocol Address IPv4 |Internet Protocol Address IPv6 |Unix domain socket end point Address |
|Media Access Control Address MAC |Hostname RFC 952 |Hostname RFC 1123 |
|HTML Tags |HTML Encoded |URL Encoded |
|Directory |Alias Validators and Tags |Non standard validators |
|Panics | | |

***

## 自定义标签

```go
// Structure
type Test struct {
	TestField string `validate:"yourtag"`
}

t := &Test{
	TestField: "Test"
}

validate := validator.New()
validate.RegisterValidation("yourtag", validators.NotBlank)
```
***

## 跨字段验证

```
- eqfield
- nefield
- gtfield
- gtefield
- ltfield
- ltefield
- eqcsfield
- necsfield
- gtcsfield
- gtecsfield
- ltcsfield
- ltecsfield
```
**example**
```go
type Inner struct {
	StartDate time.Time
}

type Outer struct {
	InnerStructField *Inner
	CreatedAt time.Time      `validate:"ltecsfield=InnerStructField.StartDate"`
}

now := time.Now()

inner := &Inner{
	StartDate: now,
}

outer := &Outer{
	InnerStructField: inner,
	CreatedAt: now,
}

errs := validate.Struct(outer)

// NOTE: when calling validate.Struct(val) topStruct will be the top level struct passed
//       into the function
//       when calling validate.VarWithValue(val, field, tag) val will be
//       whatever you pass, struct, field...
//       when calling validate.Field(field, tag) val will be nil
```

## 逗号和管道

**请使用0x2C代替逗号","**
```go
type Test struct {
	Field `validate:"excludesall=,"`    // BAD! Do not include a comma.
	Field `validate:"excludesall=0x2C"` // GOOD! Use the UTF-8 hex representation.
}
```
**请使用0x7C代替管道"|"**
```go
type Test struct {
	Field `validate:"excludesall=|"`    // BAD! Do not include a a pipe!
	Field `validate:"excludesall=0x7C"` // GOOD! Use the UTF-8 hex representation.
}
```

## tag说明
***
### Comparisons:

| tag | Description |
| -- |  -------   |
| eq  |	Equals      |
| gt  |	Greater than|
| gte |	Greater than or equal |
| lt  |	Less Than   |
|lte  |	Less Than or Equal |
|ne	  |Not Equal    |
***
### Fields

| tag | Description |
| -- |  --  | 
| eqcsfield | 	Field Equals Another Field (relative) | 
| eqfield	| Field Equals Another Field |
| fieldcontains	| NOT DOCUMENTED IN doc.go |
| fieldexcludes	| NOT DOCUMENTED IN doc.go |
| gtcsfield	| Field Greater Than Another Relative Field |
| gtecsfield	| Field Greater Than or Equal To Another Relative Field |
| gtefield	| Field Greater Than or Equal To Another Field |
| gtfield	| Field Greater Than Another Field |
| ltcsfield	| Less Than Another Relative Field |
| ltecsfield	| Less Than or Equal To Another Relative Field |
| ltefield	| Less Than or Equal To Another Field |
| ltfield	| Less Than Another Field |
| necsfield	| Field Does Not Equal Another Field (relative) |
| nefield	| Field Does Not Equal Another Field |
****
### Network

| tag | Description |
| -- |  -------   |
| cidr |	Classless Inter-Domain Routing CIDR |
| cidrv4 |	Classless Inter-Domain Routing CIDRv4 |
| cidrv6 |	Classless Inter-Domain Routing CIDRv6 |
| datauri |	Data URL |
| fqdn |	Full Qualified Domain Name (FQDN) |
| hostname |	Hostname RFC 952 |
| hostname_port |	HostPort |
| hostname_rfc1123 |	Hostname RFC 1123 |
| ip |	Internet Protocol Address IP |
| ip4_addr |	Internet Protocol Address IPv4 |
| ip6_addr |	Internet Protocol Address IPv6 |
| ip_addr |	Internet Protocol Address IP |
| ipv4 |	Internet Protocol Address IPv4 |
| ipv6 |	Internet Protocol Address IPv6 |
| mac |	Media Access Control Address MAC |
| tcp4_addr |	Transmission Control Protocol Address TCPv4 |
| tcp6_addr |	Transmission Control Protocol Address TCPv6 |
| tcp_addr |	Transmission Control Protocol Address TCP |
| udp4_addr |	User Datagram Protocol Address UDPv4 |
| udp6_addr |	User Datagram Protocol Address UDPv6 |
| udp_addr |	User Datagram Protocol Address UDP |
| unix_addr |	Unix domain socket end point Address |
| uri| URI String |
| url |	URL String |
| url_encoded |	URL Encoded |
| urn_rfc2141 |	Urn RFC 2141 String |
***
### String

| tag | Description |
| -- |  -------   | 
| alpha |	Alpha Only |
| alphanum |	Alphanumeric |
| alphanumunicode |	Alphanumeric Unicode |
| alphaunicode |	Alpha Unicode |
| ascii |	ASCII |
| contains |	Contains |
| containsany |	Contains Any |
| containsrune |	Contains Rune |
| endswith |	Ends With |
| lowercase |	Lowercase |
| multibyte |	Multi-Byte Characters |
| number |	NOT DOCUMENTED IN doc.go |
| numeric |	Numeric |
| printascii |	Printable ASCII |
| startswith |	Starts With |
| uppercase |	Uppercase |
***

### Format

| tag | Description |
| -- |  -------   |  
| base64 |	Base64 String |
| base64url |	Base64URL String |
| btc_addr |	Bitcoin Address |
| btc_addr_bech32 |	Bitcoin Bech32 Address (segwit) |
| datetime |	Datetime |
| e164 |	e164 formatted phone number |
| email |	E-mail String |
| eth_addr |	Ethereum Address |
| hexadecimal |	Hexadecimal String |
| hexcolor |	Hexcolor String |
| hsl |	HSL String |
| hsla |	HSLA String |
| html |	HTML Tags |
| html_encoded |	HTML Encoded |
| isbn |	International Standard Book Number |
| isbn10 |	International Standard Book Number 10 |
| isbn13 |	International Standard Book Number 13 |
| json |	JSON |
| latitude |	Latitude |
| longitude |	Longitude |
| rgb |	RGB String |
| rgba |	RGBA String |
| ssn |	Social Security Number SSN |
| uuid |	Universally Unique Identifier UUID |
| uuid3 |	Universally Unique Identifier UUID v3 |
| uuid3_rfc4122 |	Universally Unique Identifier UUID v3 RFC4122 |
| uuid4	Universally | Unique Identifier UUID v4 |
| uuid4_rfc4122 |	Universally Unique Identifier UUID v4 RFC4122 |
| uuid5	Universally | Unique Identifier UUID v5 |
| uuid5_rfc4122 |	Universally Unique Identifier UUID v5 RFC4122 |
| uuid_rfc4122 |	Universally Unique Identifier UUID RFC4122 |
***

### Other

| tag | Description |
| -- |  -------   |
| dir |	Directory
| endswith |	Ends With |
| excludes |	Excludes |
| excludesall |	Excludes All |
| excludesrune |	Excludes Rune |
| file |	File path |
| isdefault |	Is Default |
| len |	Length |
| max |	Maximum |
| min |	Minimum |
| oneof |	One Of |
| required |	Required |
| required_if |	Required If |
| required_unless |	Required Unless |
| required_with |	Required With |
| required_with_all |	Required With All |
| required_without |	Required Without |
| required_without_all |	Required Without All |
| excluded_with |	Excluded With |
| excluded_with_all |	Excluded With All |
| excluded_without |	Excluded Without |
| excluded_without_all |	Excluded Without All |
| unique |	Unique |
***
## 常用运算符

### Skip Field
```go
//告诉验证器跳过此struct字段，在忽略嵌入式结构被验证时很方便

Usage: -
```

### Or Operator
```go
//告诉验证器跳过此struct字段，在忽略嵌入式结构被验证时很方便

Usage: |
```

### Omit Empty
```go
//允许条件验证，例如：当字段的值未被设置时，则其他的验证不会运行
//如果设置了值，则其他的验证会运行

Usage: omitempty
```

### Dive
```go
// 告诉验证器可以进入第二级或者多级的slice array 和 map

Usage: dive

Example #1

[][]string with validation tag "gt=0,dive,len=1,dive,required"
// gt=0 will be applied to []
// len=1 will be applied to []string
// required will be applied to string

Example #2

[][]string with validation tag "gt=0,dive,dive,required"
// gt=0 will be applied to []
// []string will be spared validation
// required will be applied to string
```

### Keys & EndKeys

```go
// 和dive一起使用，可以在keys和EndKeys键之间设置使用于map的键

Usage: dive,keys,othertagvalidation(s),endkeys,valuevalidationtags

Example #1

map[string]string with validation tag "gt=0,dive,keys,eg=1|eq=2,endkeys,required"
// gt=0 will be applied to the map itself
// eg=1|eq=2 will be applied to the map keys
// required will be applied to map values

Example #2

map[[2]string]string with validation tag "gt=0,dive,keys,dive,eq=1|eq=2,endkeys,required"
// gt=0 will be applied to the map itself
// eg=1|eq=2 will be applied to each array element in the the map keys
// required will be applied to map values
```

### Required 
```go
// 确保字段包括int string slice pointer interface channel function 不为空

Usage: required
```

### Maximum 
```go
// 确保int类型的值小于等于设置的值
// 确保string类型的值的长度小于等于设置的值
// 确保slice array map 类型的值的条目小于等于设置的值

Usage: max=10
```

### Minimum  
```go
// 确保int类型的值大于等于设置的值
// 确保string类型的值的长度大于等于设置的值
// 确保slice array map 类型的值的条目大于等于设置的值

Usage: min=10
```

### Equals
```go
// 确保int string类型的值等于设置的值
// 确保slice array map 类型的值的条目等于设置的值

Usage: eq=10
Usage: eq="name"
```

### Not Equal
```go
// 确保int string类型的值不等于设置的值
// 确保slice array map 类型的值的条目不等于设置的值

Usage: ne=10
Usage: ne="name"
```

### One Of
```go
// 确保int string类型的值等于设置的一系列值中的一个

Usage: oneof=red green
       oneof=5 7 9
```

### Greater Than
```go
// 确保int类型的值大于设置的值
// 确保string类型的值的长度大于设置的值
// 确保slice array map 类型的值的条目大于设置的

Usage: gt=10

// 另外还有大于等于 Greater Than or Equal

Usage: gte=10
```

### Less Than 
```go
// 确保int类型的值小于设置的值
// 确保string类型的值的长度小于设置的值
// 确保slice array map 类型的值的条目小于设置的

Usage: lt=10

// 另外还有大于等于 Less Than or Equal

Usage: lte=10
```

### Field Equals Another Field

```go
// 确保值等于同一结构体的其他字段的值

Example #1
Usage: eqfield=ConfirmPassword

Example #2
validate.VarWithValue(password, confirmpassword, "eqfield")

// 当其他字段是结构体时，可以这样写
Usage: eqcsfield=InnerStructField.Field)

// 另外还有
// 确保不等于同一结构体的其他字段的值
// 其用法和上面的一样，tag不一样: necsfield
```

### Field Greater Than Another Field

```go
// 仅对numbers 和 time.Time 有效
// 确保值大于统一结构体中的另外字段的值

Example #1
// Validation on End field using:
validate.Struct Usage(gtfield=Start)

Example #2

// Validating by field:
validate.VarWithValue(start, end, "gtfield")

// 同样的也可以大于等于
// Field Greater Than or Equal To Another Field 
// tag为：gtefield  用法和上面一样

// 当同一结构体中的另外字段为结构体时，
// 也可以和这个二级结构体中的字段进行比较
//  tag为：gtcsfield(大于)  gtecsfield(大于等于)

Usage: gtcsfield=InnerStructField.Field
Usage: gtecsfield=InnerStructField.Field
```

### Less Than Another Field 
```go
// 这块内容可以比较上面的Field Greater Than Another Field内容来理解
// 实现的功能是相反的： 小于 和 小于等于
// tag:  ltfield (小于) ltefield(小于等于)  ltcsfield(跨字段小于) ltecsfield(跨字段小于等于)
// 使用方式和上面完全相同，不同的只是tag
```

### Unique 
```go
// 确保array和slice中没有重复
// 确保 map 的 value 没有重复
// 确保 struct 中的字段的value 没有重复

// For arrays, slices, and maps:
Usage: unique

// For slices of struct:
Usage: unique=field
```

### Alpha Only
```go
// 确保字符串值仅包含ASCLL字母字符

Usage: alpha
```

### Alphanumeric 
```go
// 确保字符串仅包含ASCLL字母数字字符

Usage: alphanum
```

### Alpha Unicode
```go
// 确保字符串值仅包含Unicode 字母字符

Usage: alphaunicode
```

### Alphanumeric Unicode
```go
// 确保字符串值仅包含Unicode字母数字字符

Usage: alphanumunicode
```

### Numeric 
```go
// 确保字符串值只包含基本的数字，不包含指数等等
// 对于整数和浮点数返回true

Usage: numeric
```

### Hexadecimal String 
```go
// 确保字符串值包含有效的十六进制
// 对于整数和浮点数返回true

Usage: hexadecimal

// 另外的，像如此这样的功能有很多，这里举例几个，具体用法查看官方文档
// 确保字符串值包含有效的十六进制颜色（包括井号）
// 确保字符串值包含有效的rgb颜色
// 确保字符串值包含有效的rgba颜色
// 确保字符串值包含有效的hsl颜色
// 确保字符串值包含有效的hsla颜色
```

### E-mail String 
```go
// 确保字段值是有效的邮箱
// 需要注意的是只是确保的是邮箱格式，如果想要更加规范的邮箱格式，可以使用正则

Usage: email
```

### File path
```go
// 确保字段值是有效的文件路径

Usage: file
```

### URL String
```go
// 确保字段值是有效的URL
// 字段值必须要包含http:// or rtmp://

Usage: url
```

### URL String
```go
// 确保字段值是有效的URI

Usage: uri
```

### Contains 
```go
// 确保字段字符串含有指定的子字符串

Usage: contains=@
```

### Excludes  
```go
// 确保字段字符串不含有指定的子字符串

Usage: excludes=@
```

### Starts With  
```go
// 确保字段字符串是以指定的字符串开头的

Usage: startswith=hello
```

### Ends With  
```go
// 确保字段字符串是以指定的字符串结尾的

Usage: endswith=goodbye
```

### HTML Tags
```go
// 确保字符串是一个HTML element tag
// 例如：<base> <head> <link> <meta> <style> <title> 等

Usage: html
```
***

## 范围比较验证
请参考上面文档`tags说明`的`Comparisons`部分

范围验证：切片、数组和map、字符串，验证其长度；数值验证大小范围

对于time.Time类型，将和time.Now.UTC() 进行比较验证
* lte：小于等于参数值，`validate:"lte=3"` (小于等于3)
* gte：大于等于参数值，`validate:"lte=120,gte=0"` (大于等于0小于等于120)
* lt：小于参数值，`validate:"lt=3" `(小于3)
* gt：大于参数值，`validate:"lt=120,gt=0" `(大于0小于120)
* len：等于参数值，`validate:"len=2"` (长度等于2)
* max：最大值，小于等于参数值，`validate:"max=20" `(小于等于20)
* min：最小值，大于等于参数值，`validate:"min=2,max=20"` (大于等于2小于等于20)

Example:
```go
type User struct {
    Name string `json:"name" validate:"min=0,max=35"`
    Age  unit8  `json:"age" validate:"lte=90,gte=0"`
}
```
***

## 字符串验证
请参考上面文档`tags说明`的`strings`部分

范围验证: 切片、数组和map、字符串，验证其长度；数值，验证大小范围

* contains：包含参数子串，`validate:"contains=tom"` (字段的字符串值包含tom)
* excludes：包含参数子串，`validate:"excludes=tom"` (字段的字符串值不包含tom)
* startswith：以参数子串为前缀，`validate:"startswith=golang"`
* endswith：以参数子串为后缀，`validate:"startswith=world"`

Example:
```go
type User struct { 
    Name string `validate:"contains=tom"` 
    Age int `validate:"min=1"`
}
```
***

## 验证字段
请参考上面的`·tag说明`的`Fields`部分

* eqcsfield：跨不同结构体字段验证，比如说 Struct1 Filed1，与结构体Struct2 Field2相等，

Example:
```go
type Struct1 struct {
    Field1 string `validate:eqcsfield=Struct2.Field2`
    Struct2 struct {
        Field2 string 
    }
}
```
* necsfield：跨不同结构体字段不相等

* eqfield：同一结构体字段验证相等，最常见的就是输入2次密码验证

Example:
```go
type User struct { 
    Name string `validate:"lte=4"` 
    Age int `validate:"min=20"` 
    Password string `validate:"min=10"`
    Password2 string `validate:"eqfield=Password"`
}
```

* nefield：同一结构体字段验证不相等

Example:
```go
type User struct {
    Name string `validate:"lte=4"` 
    Age int `validate:"min=20"` 
    Password string `validate:"min=10,nefield=Name"`
}
```

* gtefield：大于等于同一结构体字段，`validate:"gtefiled=Field2"`

* ltefield：小于等于同一结构体字段
***

## 验证单个字段变量值
Example:
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	validate := validator.New()

	var boolTest bool
	err := validate.Var(boolTest, "required")
	if err != nil {
		fmt.Println(err) // error
	}
	
	var stringTest string = ""
	err = validate.Var(stringTest, "required")
	if err != nil {
		fmt.Println(err) // error
	}

	var emailTest string = "test@136.com"
	err = validate.Var(emailTest, "email")
	if err != nil {
		fmt.Println(err) // success
	}

	var ipTest string = "127.0.0.0"
	err = validate.Var(ipTest, "ip")
	if err != nil {
		fmt.Println(err) // success
	}

	fmt.Println("end")
}
```
> 上面的代码大致分为两步：
> + 创建一个实例
> + 使用自带标签检测变量
***

## 网络验证
请参考上面`tag说明`的`Network`

* ip：字段值是否包含有效的IP地址，validate:"ip"
* ipv4：字段值是否包含有效的ipv4地址，validate:"ipv4"
* ipv6：字段值是否包含有效的ipv6地址，validate:"ipv6"
* uri：字段值是否包含有效的uri，validate:"uri"
* url：字段值是否包含有效的uri，validate:"url"
***

## 验证结构体
``` go
package main

import (
	"fmt"
    "github.com/go-playground/validator/v10"
)

type Person struct {
    Name    string  `validate:"check"`
}

// 自定义验证器
func CustomValidator(f validator.FieldLevel) bool {
	fmt.Println(f.Field().String())
    return f.Field().String() == "liu"
}

func main() {
    person := &Person {
        Name : "liucheng",
    }
	
    // new一个实例
    v := validator.New()

    // 注册自定义tag
    v.RegisterValidation("check", func(f validator.FieldLevel) bool {
		fmt.Println(f.Field().String())
		return f.Field().String() == "liu"
	})

    // 检测结构体
    err := v.Struct(person)
	if err != nil {
		fmt.Println("struct filed value error:", err) // 结果为error
		return
	}
}
```
以上就是一个检测结构体的基本例子，需要注意的是结构中的字段首字母需要大写，即为导出字段才能检测的到，另外本例使用的是自定义tag所以需要注册tag,如果使用的是本文之前所列出的自带tag是不需要这一步的。
> 上面的代码大致分为三步：
> + 创建一个实例
> + 注册自定义标签tag
> + 检测结构体字段
***

## 验证slice
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	sliceOne := []string{"zhangsan", "lisi", "wangwu", "chenliu"}

	validate := validator.New()
	err := validate.Var(sliceOne, "max=5,dive,min=6")
	if err != nil {
		fmt.Println("err:", err) // error cliceOne[1]="lisi"的length小于6
	}

	sliceTwo := []string{}
	err = validate.Var(sliceTwo, "min=4,dive,required")
	if err != nil {
		fmt.Println(err)
		// error
		// sliceTwo[] 条目数为0，小于4 即error
		// sliceTwo[] 的value 为 "" 不满足required 即error
	}
}
```
本例子的代码实现思想和`验证单个字段`一致，需要注意的是，当slice是二维或者多维时，使用相应层数的dive就可以进入到对应的维数中了。
***

## 验证map
可以使用keys&Endkeys来配合div使用，从而准确的验证map的key和value

```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	var mapone map[string]string

	mapone = map[string]string{"one": "zhangsan", "two": "lisi", "three": ""}

	validate := validator.New()
	err := validate.Var(mapone, "gte=3,dive,keys,eq=1|eq=2,endkeys,required")
	if err != nil {
		fmt.Println(err)
		// error
		// "one" 不满足 eq=1|eq=2
		// "two" 不满足 eq=1|eq=2
		// "three" 不满足 eq=1|eq=2
		// "three" 不满足 required
	}
}
```
**嵌套map怎么验证呢？**

和上面的slice差不多，使用多个dive就行

Example:
```go
var maptwo map[[3]string]string{}
validate.Var(maptwo, "gte=3,dive,keys,dive,eq=1|eq=3,endkeys,required")
```

## 通过字段tag自定义函数
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	Name string `validate:"required,yourTag"`
	// 注意：required和yourTag之间不能有空格，否则panic
	// 注意：Name首字母需要大写，不然检测不到

	Age  uint8  `validate:"gte=0,lte=80"`
	// 注意：gte=0和lte=80之间不能有空格，否则panic
}

var validate *validator.Validate

func main() {
	validate = validator.New()

	//注册自定义函数，前一个参数是struct里tag自定义，后一个参数是自定义的函数
	validate.RegisterValidation("yourTag", CustomerValidationFunc)

	user := &User{
		Name: "zhangsan",
		Age:  81,
	}

	err := validate.Struct(user)
	if err != nil {
		fmt.Printf("Err(s):\n%v\n", err)
		// error
		// Name 不匹配 "zhangsan" != "ZhangSan"
		// Age 不匹配 81 不在 0~80之间
	}

	user.Name = "ZhangSan"
	user.Age = 18
	err = validate.Struct(user)
	if err != nil {
		fmt.Printf("Err(s):\n%v\n", err)
	}
}

// 自定义函数
func CustomerValidationFunc(f1 validator.FieldLevel) bool {
    // f1 包含了字段相关信息
    // f1.Field() 获取当前字段信息
    // f1.Param() 获取tag对应的参数
    // f1.FieldName() 获取字段名称

	return f1.Field().String() == "ZhangSan"
}
```

## 自定义函数-直接注册函数

不通过字段tag自定义函数，直接注册函数。


```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	FirstName      string `json:"firstname"`
	LastName       string `json:"lastname"`
	Age            uint8  `validate:"gte=0,lte=130"`
	Email          string `validate:"required,email"`
	FavouriteColor string `validate:"hexcolor|rgb|rgba"`
}

var validate *validator.Validate

func main() {
	validate = validator.New()

	validate.RegisterStructValidation(UserStructLevelValidation, User{})

	user := &User{
		FirstName:      "",
		LastName:       "",
		Age:            30,
		Email:          "TestFunc@126.com",
		FavouriteColor: "#000",
	}

	err := validate.Struct(user)
	if err != nil {
		fmt.Println(err)
	}
}

func UserStructLevelValidation(sl validator.StructLevel) {
	user := sl.Current().Interface().(User)

	if len(user.FirstName) == 0 && len(user.LastName) == 0 {
		sl.ReportError(user.FirstName, "FirstName", "firstname", "firstname", "") // error
		sl.ReportError(user.LastName, "LastName", "lastname", "lastname", "") //error
	}
}
```

## 两变量字段进行比较

```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	field1 := "liu"
	field2 := "niu"

	validate := validator.New()

	err := validate.VarWithValue(field1, field2, "nefield")
	if err != nil {
		fmt.Println(err)  // success 
	}

	err = validate.VarWithValue(field1, field2, "eqfield")
	if err != nil {
		fmt.Println(err)   // error "liu" != "niu"
	}
}
```
