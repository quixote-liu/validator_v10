# validator

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

本文档根据上面的两个文档进行参考而写，观看本文档可以查看上面两个文档
***

## 文档概述

在平常开发中，特别是web应用开发中，为了验证输入字段的合法性，都会做出一些验证操作。比如对用户提交的表单字段进行验证，或者对请求的API接口字段进行验证，验证字段的合法性，保证输入字段值的安全，防止用户的恶意请求。一般的做法是用正则表达式，一个字段一个字段的进行验证，这样比较繁琐，而validator这个验证组件就可以很大程度的简化验证过程。

github.com/go-playground/validator/v10 库提供了非常丰富和强大的检验功能，它的主要作用就是用来检验参数，核心的检验方式是通过tag来检验，tag相当于是一个检验标识，可以将其看作是检验器的延申，tag绑定的检验器（检验函数）完成检验的工作。使用库中自带tag不需要自己设置检验器（检验函数），这个库中提供了很多检验tag，使用者可以根据自己的需求来选择。
***

## 功能介绍
****

### 使用tag的前置说明

* 如果想tag中含有逗号","，请使用0x2C代替，因为逗号","是validator默认tag分割符
* 如果想tag中含有竖线"|"，请使用0x7C代替，因为竖线"|"是validator的或运算符
* 横线"-"标识跳过此字段
* required: 表示该字段为必须输入，且不能为默认值
* omitempty: 如果字段没有输入值，则忽略检测，如果输入了值则检测
* 对结构体中的字段使用tag时，必须保证字段的首字母大写即为导出字段，否则将会检测不到，但是不报错。
* 连续的tag之间不能有空格，例如：min=0, max=5，否则会报错
*******

### 范围比较验证

范围验证:切片、数组和map、字符串，验证其长度；数值验证大小范围

* lt: 小于参数值，`validate:"lt=3" `(小于3)
* lte: 小于等于参数值，`validate:"lte=3"` (小于等于3)
* gt：大于参数值，`validate:"lt=120,gt=0"` (大于0小于120)
* gte：大于等于参数值，`validate:"lte=120,gte=0" `(大于等于0小于等于120)
* max：最大值，小于等于参数值，`validate:"max=20"` (小于等于20)
* min：最小值，大于等于参数值，`validate:"min=2,max=20"` (大于等于2小于等于20)
* len：等于参数值，`validate:"len=2"`
* ne：不等于，`validate:"ne=2"` (不等于2)
* oneof：只能是列举出的值其中一个，这些值必须是数值或字符串，以空格分隔，如果字符串中有空格，将字符串用单引号包围，`validate:"oneof=red green"`

Example:
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	Name    string    `validate:"required,max=10"`
	Age     int       `validate:"required,min=18,max=80"`
	Phone   string    `validate:"len=11"`
	Hobby   []string  `validate:"required,gt=0,lte=3"`
	Company string    `validate:"oneof= tongfang youyun"`
}

func main() {
	user := &User {
		Name: "ZhangYaJun",
		Age:  20,
		Phone: "123456789",
		Hobby: []string{
			"write code",
			"write code",
			"write code",
			"write code",
		},
		Company: "hubeidaxue",
	}
	fmt.Println("Start")

	// 创建一个实例
	validate :=  validator.New()

	// 检测结构体
	err := validate.Struct(user)
	if err != nil {
		fmt.Println(err)
		// error_1  User.Phone   failed on the 'len' tag
		// error_2  User.Hobby   failed on the 'lte' tag
		// error_3  User.Company failed on the 'oneof' tag
	}

	fmt.Println("End")
}

// 本例是tag的基础运用，tag就相当于传统的正则表达式一样，将自己想要条件设置在tag中，然后检测字段是否合乎要求，不过比起正则表达式validator显然要简便的多
```
可以从上面例子看到大致的检测流程
1. 定义一个validate的实例 用到 validator.New()
2. 检测带有tag的结构体， 用到 validate.Struct(user)
**********

### 字符串验证

可以从上面的例子看到，范围验证只能验证字符串的长度，如果想要对字符串稍微复杂一点的验证可以使用到其他字符串验证tag，此处只举一些简单的tag例子，更多的tag例子请看validator_v10参考文档，用法都是千篇一律的，至于更加复杂和精确的检验可以自己定义检测tag(本文后面会讲到)。

* contains：包含参数子串，validate:"contains=tom" (字段的字符串值包含tom)
* excludes：包含参数子串，validate:"excludes=tom" (字段的字符串值不包含tom)
* startswith：以参数子串为前缀，validate:"startswith=golang"
* endswith：以参数子串为后缀，validate:"endswith=world"

Example:

```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type Client struct {
	Name    string    `validate:"required,contains=HuBei,excludes=DaXue"`
	Linkman string    `validate:"startswith=Mr.|startswith=Ms."`
	Address string    `validate:"endswith=(China)"`
}

func main() {
	client := &Client {
		Name: "HuBeiDaXue",
		Linkman: "Mr.Zhang",
		Address: "HuBeiChina",
	}
	fmt.Println("Start")

	// 创建一个实例
	validate :=  validator.New()

	// 检测结构体
	err := validate.Struct(client)
	if err != nil {
		fmt.Println(err)
		// error_1  Client.Name   failed on the 'excludes' tag
		// error_2  Client.Address   failed on the 'endswith' tag
	}

	fmt.Println("End")
}
```
本例是检验字符串的简单运用，需要注意的是使用contains和excludes时，必须要带上required标签，不然会报错

检测的大致流程和使用的方法和范围检测一样，这里不再赘述
***********

### 字段验证

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

* ltefield：小于等于同一结构体字段，`validate:"ltefield=Field2"`

Example:
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type contact struct {
	Phone  string
	Email  string
}

type User struct { 
    Name       string  `validate:"lte=20"` 
    Age        int     `validate:"min=20"`
	Phone      string  `validate:"necsfield=Contact.Phone"`
	Email      string  `validate:"eqcsfield=Contact.Email"`
	Contact    contact
    Password   string  `validate:"min=10,nefield=Name"`
    Password2  string  `validate:"eqfield=Password"`
}

func main() {
	user := &User {
		Name: "ZhangYaJun",
		Age: 20,
		Phone: "12244448888",
		Email: "132@fox.com",
		Contact: contact{
			Phone: "12244448888",
			Email: "132@fox.com",
		},
		Password: "111222333444",
		Password2: "111444777888",
	}
	fmt.Println("Start")

	// 创建一个实例
	validate :=  validator.New()

	// 检测结构体
	err := validate.Struct(user)
	if err != nil {
		fmt.Println(err)
		// error_1  User.Phone  'Phone' failed on the 'necsfield' tag
		// error_2  User.Password2   'Password2' failed on the 'eqfield' tag
	}

	fmt.Println("End")
}
// 上面的例子是字段验证的基本运用，起实现流程和使用方法和之前的几个例子一样。
```
***

### 网络验证

网络验证和普通的字段验证一样，不过使用的tag不同，关于网络的tag一般有这么几个(更详细的请看validator文档)：

* ip：字段值是否包含有效的IP地址，validate:"ip"
* ipv4：字段值是否包含有效的ipv4地址，validate:"ipv4"
* ipv6：字段值是否包含有效的ipv6地址，validate:"ipv6"
* uri：字段值是否包含有效的uri，validate:"uri"
* url：字段值是否包含有效的uri，validate:"url"

Example:
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type contact struct {
	Phone  string
	Email  string
}

type NetWork struct { 
   IP   string    `validate:"ip"` 
   IPV4 string    `validate:"ipv4"`
   IPV6 string    `validate:"ipv6"`
   URL  string    `validate:"url"`
   URI  string    `validate:"uri"`
}

func main() {
	net := &NetWork {
		IP:   "127.0.0.0",
		IPV4: "127.255.255.254",
		IPV6: "2001:0D12:0000:0000:02AA:0987:FE29:9871",
		URL:  "https://github.com/go-playground/validator",
		URI:  "http://127.0.0.1:8080/cmd_helloworld/?name=guowuxin",
	}
	fmt.Println("Start")

	// 创建一个实例
	validate :=  validator.New()

	// 检测结构体
	err := validate.Struct(net)
	if err != nil {
		fmt.Println(err)
		// success
	}

	fmt.Println("End")
}
```
****

### 验证单个字段变量值

我们上面的例子全部使用的是结构体，但是如果现在需要有单个字段需要做验证怎么办？可以直接看下面的代码，

Example：
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
可以看到验证单个字段也是使用的tag来检测，在validator中tag就相当于是检测器的延申，不过使用的方法是validate.Var()，此方法的具体解释可以查看validator文档，这里不再赘述。
*****

### 两变量字段进行比较

相比于上个例子，这个例子进行了稍微的拓展，主要是介绍了VarWithValue()方法的作用和使用方式。请看代码

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
从上面可以看出，两个字段之间进行比较还是通过tag进行比较

****
### 验证slice

验证golang中常用类型的slice，由于验证结构体上面的几个例子都是验证的结构体，所以就不重复写验证结构体了。验证slice 可以配上dive来很好的验证key-value，直接看代码更加清楚。

Example:
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
***

### 验证map

上面验证了slice,和这里验证map的方式差不多，不过从slice和map的区别来看，关键在于key值的不同，slice的key是固定的int类型的index,但是map的key是变化的string类型，对于这种情况可以使用keys&Endkeys来配合dive使用，从而准确的验证map的key-value,废话不多说，上代码

Example:
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
// 本列对map的key值设置了eq=1|eq=2的条件设置，对value设置了required的条件。
```
**嵌套map怎么验证呢？**

很简单使用多个dive就行，想进入哪一级就使用可以连续使用多个dive，slice的嵌套也可以这样验证。

Example:
```go
var maptwo map[[3]string]string{}
validate.Var(maptwo, "gte=3,dive,keys,dive,eq=1|eq=3,endkeys,required")
```
*****

### 通过字段tag自定义函数
当validator中的tag不满足你的需求怎么办？你可以自定义一个tag，然后这个对这个tag注册一个满足你的需求的检测函数，这样这个tag就可以向上面的例子中的一样的使用了。看代码。

Example:
```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	Name string `validate:"required,yourTag"`
	Age  uint8  `validate:"gte=0,lte=80"`
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
    // 更多的信息请看validator文档

	return f1.Field().String() == "ZhangSan"
}
```

从上面的例子可以看到，注册自定义tag的流程非常简便和简洁
1. 创建一个自定义tag名，（可以先直接写在需要检测的结构体中）
2. 创建一个检测函数，检测函数类型：func(f1 validator.FieldLevel) bool 
3. 将检测函数注册到tag中：validate.RegisterValidation("yourTag", yourTestFunc)

从这里可以也可以理解validator的核心思想就是使用tag去完成检测，很多操作都是围绕tag来做的，这样可以让程序变得更加简洁和干净，用起来很利索，效率高。
****

### 自定义函数-直接注册函数

看到这里也理解到了使用tag检测的优点，但是如果仔细思考的话就会发现tag也有一些局限性，比如说我的结构体很大很复杂需求很多，而validator中的自带tag又满足不了我的复杂需求，如果使用自定义tag,就要一个一个的为每一个字段设置检测函数然后进行注册，这就太麻烦了。所以这里可以使用自定义函数-直接注册函数，跳过tag来对结构体进行检测。直接上代码：

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

看完代码可以知道，我们使用了RegisterStructValidation()方法来对一个结构体User{}注册了一个检测函数UserStructLevelValidation(sl validator.StructLevel)，还是老规矩，方法的具体功能请看参考文档，这里不赘述。这个检测函数就可以对经过参数sl对这个结构体的字段进行操作，我们就可以写一些满足自己需求的条件。这样就达到了检测的需要。

## 总结

validator 的类型和方法有不少，我个人认为可以围绕使用最多的类型也就是type FieldLevel 和 type StructLevel 进行展开。

FieldLevel面向简单单个字段验证，当需求不多时，可以考虑使用，自定义一个tag，然后可以再检验函数中通过FieldLevel类型的参数获取字段的一些信息，从而进行操作达到检测的目的。

StructLevel面向的时比较稍微复杂一点的需求，当然需求简单时可以使用，在代码量上相差不大，而在功能上对比FieldLevel更加灵活一些，可以随便访问到结构体的任何字段，在此基础上进行操作实现复杂的需求。

对于交叉使用，因为在开发中遇到的大多数是结构体，可以运用面向对象的思维，给这个结构体增加检测的方法去对valitor的方法进行封装，这样看起来很方便使用起来效率也会很高。

还有关于自定义的tag，本人认为应该可以更加灵活的去理解它，比如说这个自定义的tag和普通的validator的tag并没有什么不同，所以我认为凡是在使用到tag string 的方法中应该都可以运用的到。

限于笔者的水平，未能玩到更多方法和类型。文档中的一些地方写的或许有些问题还请多多指教。