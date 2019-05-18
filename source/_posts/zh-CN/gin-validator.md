---
title: gin中是如何使用validator?
lang: zh-CN
date: 2019-05-18 16:32:17
tags: 
  - golang
  - gin
---

## 先介绍下validator
### validator v8与v9
gin中使用的validator是v8版本，v8与v9版本区别比较大。
其中一个区别是v8版本没有指定默认的TagName，需要在每次声明validator实例的时候配置；在v9中设置了默认的TagName就是validate。我们的gin框架在使用的时候设置的TagName为"binding"。
<!-- more -->
validator中给的例子代码
```go
    var validate *validator.Validate
    config := &validator.Config{TagName: "validate"}
    validate = validator.New(config)
```

gin中设置的TagName，文件`gin/binding/default_validator.go`
```go
func (v *defaultValidator) lazyinit() {
	v.once.Do(func() {
		config := &validator.Config{TagName: "binding"}
		v.validate = validator.New(config)
	})
}
```
这也就是为啥gin框架里面example中的定义的struct都是binding开头
```go
type Booking struct {
	CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
	CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn" time_format:"2006-01-02"`
}
```
其他区别例如实例化不同等

### validator使用的tag
validator使用的struct的tag有以下这么多，其中有一些是通过正则表达式实现类型判断的，例如email、rgb、uuid等；有一些是通过第三方库实现的，例如ipv4、cidr等。
```go
// validator/baked_in.go
bakedInValidators = map[string]Func{
		"required":         hasValue,
		"isdefault":        isDefault,
		"len":              hasLengthOf,
		"min":              hasMinOf,
		"max":              hasMaxOf,
		"eq":               isEq,
		"ne":               isNe,
		"lt":               isLt,
		"lte":              isLte,
		"gt":               isGt,
		"gte":              isGte,
		"eqfield":          isEqField,
		"eqcsfield":        isEqCrossStructField,
		"necsfield":        isNeCrossStructField,
		"gtcsfield":        isGtCrossStructField,
		"gtecsfield":       isGteCrossStructField,
		"ltcsfield":        isLtCrossStructField,
		"ltecsfield":       isLteCrossStructField,
		"nefield":          isNeField,
		"gtefield":         isGteField,
		"gtfield":          isGtField,
		"ltefield":         isLteField,
		"ltfield":          isLtField,
		"alpha":            isAlpha,
		"alphanum":         isAlphanum,
		"alphaunicode":     isAlphaUnicode,
		"alphanumunicode":  isAlphanumUnicode,
		"numeric":          isNumeric,
		"number":           isNumber,
		"hexadecimal":      isHexadecimal,
		"hexcolor":         isHEXColor,
		"rgb":              isRGB,
		"rgba":             isRGBA,
		"hsl":              isHSL,
		"hsla":             isHSLA,
		"email":            isEmail,
		"url":              isURL,
		"uri":              isURI,
		"file":             isFile,
		"base64":           isBase64,
		"base64url":        isBase64URL,
		"contains":         contains,
		"containsany":      containsAny,
		"containsrune":     containsRune,
		"excludes":         excludes,
		"excludesall":      excludesAll,
		"excludesrune":     excludesRune,
		"isbn":             isISBN,
		"isbn10":           isISBN10,
		"isbn13":           isISBN13,
		"eth_addr":         isEthereumAddress,
		"btc_addr":         isBitcoinAddress,
		"btc_addr_bech32":  isBitcoinBech32Address,
		"uuid":             isUUID,
		"uuid3":            isUUID3,
		"uuid4":            isUUID4,
		"uuid5":            isUUID5,
		"ascii":            isASCII,
		"printascii":       isPrintableASCII,
		"multibyte":        hasMultiByteCharacter,
		"datauri":          isDataURI,
		"latitude":         isLatitude,
		"longitude":        isLongitude,
		"ssn":              isSSN,
		"ipv4":             isIPv4,
		"ipv6":             isIPv6,
		"ip":               isIP,
		"cidrv4":           isCIDRv4,
		"cidrv6":           isCIDRv6,
		"cidr":             isCIDR,
		"tcp4_addr":        isTCP4AddrResolvable,
		"tcp6_addr":        isTCP6AddrResolvable,
		"tcp_addr":         isTCPAddrResolvable,
		"udp4_addr":        isUDP4AddrResolvable,
		"udp6_addr":        isUDP6AddrResolvable,
		"udp_addr":         isUDPAddrResolvable,
		"ip4_addr":         isIP4AddrResolvable,
		"ip6_addr":         isIP6AddrResolvable,
		"ip_addr":          isIPAddrResolvable,
		"unix_addr":        isUnixAddrResolvable,
		"mac":              isMAC,
		"hostname":         isHostnameRFC952,  // RFC 952
		"hostname_rfc1123": isHostnameRFC1123, // RFC 1123
		"fqdn":             isFQDN,
		"unique":           isUnique,
		"oneof":            isOneOf,
		"html":             isHTML,
		"html_encoded":     isHTMLEncoded,
		"url_encoded":      isURLEncoded,
	}
```
tag中出现的dive的使用，dive一般用在slice、array、map、嵌套的struct验证中，作为分隔符表示进入里面一层的验证规则
```go
type Test struct {
	Array []string `validate:"required,gt=0,dive,max=2"`
    //slice使用gt=0代表field.len()>0，所以[]string{}复合required规则，但是不符合gt=0规则
	Map map[string]string `validate:"required,gt=0,dive,keys,max=10,endkeys,required,max=2"`
	// dive代表进入里面一层的验证 例如 a={"b":"c"}中 dive之前的required代表a是必填，大于0，
	//dive之后出现keys与endkeys之间代表验证map的keys的tag值：max=10，也就是string长度不大于10
	//后面的自然而然的是验证value了，required 必填并且stirng最大长度是2
}
```
字段之间的关系使用如下标签，例如gtfield=CheckIn代表字段值要大于CheckIn字段的值。
```go
        "eqfield":          isEqField,
        "eqcsfield":        isEqCrossStructField,
        "necsfield":        isNeCrossStructField,
        "gtcsfield":        isGtCrossStructField,
        "gtecsfield":       isGteCrossStructField,
        "ltcsfield":        isLtCrossStructField,
        "ltecsfield":       isLteCrossStructField,
        "nefield":          isNeField,
        "gtefield":         isGteField,
        "gtfield":          isGtField,
        "ltefield":         isLteField,
        "ltfield":          isLtField,
```
自定义的验证tag的使用例子，以下自定义了is-awesome标签。
```go
package main
import (
	"fmt"
	"gopkg.in/go-playground/validator.v9"
)
type MyStruct struct {
	String string `validate:"is-awesome"`
}
func ValidateMyVal(f1 validator.FieldLevel) bool {
	return f1.Field().String() == "awesome"
}
var validate *validator.Validate
func main() {
	validate = validator.New()
	validate.RegisterValidation("is-awesome", ValidateMyVal)
	s := MyStruct{String: "awe5some"}
	err := validate.Struct(s)
	if err != nil{
		fmt.Printf("Err(s):\n%+v\n", err)
	}else{
		fmt.Println("validate input")
	}
	s.String = "not-awesome"
	fmt.Println(s)
	err = validate.Struct(s)
	fmt.Println(err)
	fmt.Println(s.String == "awesome")
	if  err!=nil{
		fmt.Printf("Err(s):\n%+v\n", err)
	}else{
		fmt.Println("yoyo")
	}
}
```
自定义struct level的验证，以下例子在验证FirstName与LastName时候，为了确保他们之中必须有一个有值。留一个问题，这里面的json与gin中的json的区别？？？？；同样以下例子同时使用了嵌套的struct，在User中定义了一个指向Address的列表指针，该指针不能为nil，并且指针指向的Address值也能为nil。
```go
package main
import (
	"fmt"
	"gopkg.in/go-playground/validator.v9"
)
// User contains user information
type User struct {
	FirstName      string     `json:"fname"`
	LastName       string     `json:"lname"`
	Age            uint8      `validate:"gte=0,lte=130"`
	Email          string     `validate:"required,email"`
	FavouriteColor string     `validate:"hexcolor|rgb|rgba"`
	Addresses      []*Address `validate:"required,dive,required"` // a person can have a home and cottage...
}
// Address houses a users address information
type Address struct {
	Street string `validate:"required"`
	City   string `validate:"required"`
	Planet string `validate:"required"`
	Phone  string `validate:"required"`
}
// use a single instance of Validate, it caches struct info
var validate *validator.Validate
func main() {

	validate = validator.New()

	// register validation for 'User'
	// NOTE: only have to register a non-pointer type for 'User', validator
	// interanlly dereferences during it's type checks.
	validate.RegisterStructValidation(UserStructLevelValidation, User{})
	// build 'User' info, normally posted data etc...
	address := &Address{
		Street: "Eavesdown Docks",
		Planet: "Persphone",
		Phone:  "none",
		City:   "Unknown",
	}
	user := &User{
		FirstName:      "",
		LastName:       "11",
		Age:            45,
		Email:          "Badger.Smith@gmail.com",
		FavouriteColor: "#000",
		Addresses:      []*Address{address},
	}
	// returns InvalidValidationError for bad validation input, nil or ValidationErrors ( []FieldError )
	err := validate.Struct(user)
	if err != nil {
		// this check is only needed when your code could produce
		// an invalid value for validation such as interface with nil
		// value most including myself do not usually have code like this.
		if _, ok := err.(*validator.InvalidValidationError); ok {
			fmt.Println("err is ",err)
			return
		}
		for _, err := range err.(validator.ValidationErrors) {
			fmt.Println("err is ", err)
			fmt.Println(err.Namespace())
			fmt.Println(err.Field())
			fmt.Println(err.StructNamespace()) // can differ when a custom TagNameFunc is registered or
			fmt.Println(err.StructField())     // by passing alt name to ReportError like below
			fmt.Println(err.Tag())
			fmt.Println(err.ActualTag())
			fmt.Println(err.Kind())
			fmt.Println(err.Type())
			fmt.Println(err.Value())
			fmt.Println(err.Param())
			fmt.Println()
		}
		// from here you can create your own error messages in whatever language you wish
		return
	}
	// save user to database
}
// UserStructLevelValidation contains custom struct level validations that don't always
// make sense at the field validation level. For Example this function validates that either
// FirstName or LastName exist; could have done that with a custom field validation but then
// would have had to add it to both fields duplicating the logic + overhead, this way it's
// only validated once.
//
// NOTE: you may ask why wouldn't I just do this outside of validator, because doing this way
// hooks right into validator and you can combine with validation tags and still have a
// common error output format.
func UserStructLevelValidation(sl validator.StructLevel) {
	user := sl.Current().Interface().(User)
	if len(user.FirstName) == 0 && len(user.LastName) == 0 {
		sl.ReportError(user.FirstName, "FirstName", "fname", "fnameorlname", "")
		sl.ReportError(user.LastName, "LastName", "lname", "fnameorlname", "")
	}
	// plus can to more, even with different tag than "fnameorlname"
}
```
## gin是如何使用validator的？
1. gin中的struct的tag增加了content-type支持，分别是一下所示，其中ProtoBuf 还不支持验证
```go
var (
	JSON          = jsonBinding{}
	XML           = xmlBinding{}
	Form          = formBinding{}
	Query         = queryBinding{}
	FormPost      = formPostBinding{}
	FormMultipart = formMultipartBinding{}
	ProtoBuf      = protobufBinding{}
	MsgPack       = msgpackBinding{}
)
```
常用的就form、query、json和xml；经过测试url中的传参和body中传参（application/x-www-form-urlencoded）都可以使用form标签
```go
type Login struct {
	User     string `form:"user" json:"user" xml:"user"  binding:"required"`
	Password string `form:"password" json:"password" xml:"password" binding:"required"`
}
```
2. 还支持添加字段的默认值
```go
type Login struct {
	User     string `form:"user,default=nidaye"  binding:"required"`
	Password string `form:"password"  binding:"required"`
}
```
3. shouldbind与mustbind的区别，这里直接列出源代码解释。
```go
// MustBindWith binds the passed struct pointer using the specified binding engine.
// It will abort the request with HTTP 400 if any error ocurrs.
// See the binding package.
func (c *Context) MustBindWith(obj interface{}, b binding.Binding) (err error) {
	if err = c.ShouldBindWith(obj, b); err != nil {
		c.AbortWithError(http.StatusBadRequest, err).SetType(ErrorTypeBind)
	}
	return
}
```
4. 问gin中是如何把定义的Struct结合到浏览器传过来的数据，然后交给validator校验的呢？
以query类型为例子，文件`gin/binding/query.go`；在bind方法中通过mapForm方法绑定值到sturct类型上，具体怎么绑可以看源码，大概就是利用反射。
```go
func (queryBinding) Bind(req *http.Request, obj interface{}) error {
	values := req.URL.Query()
	if err := mapForm(obj, values); err != nil {
		return err
	}
	return validate(obj)
}
```


