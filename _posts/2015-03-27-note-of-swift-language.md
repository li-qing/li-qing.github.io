---
layout: post
title: The Swift Programming Language 笔记
category: iOS
tags: Swift
---

## The Basics

1\. Unicode字符可以用作变量/常量名 eg:

{% highlight swift %}
let π = 3.14159
{% endhighlight %}

2\. /**/采用最近匹配, 支持多层. eg:

{% highlight swift %}
/* this is the start of the first multiline comment
/* this is the second, nested multiline comment */
this is the end of the first multiline comment */
{% endhighlight %}

3\. 二进制/八进制/十进制/十六进制

{% highlight swift %}
let binaryInteger = 0b10001       // 17 in binary notation
let octalInteger = 0o21           // 17 in octal notation
let decimalInteger = 17
let hexadecimalInteger = 0x11     // 17 in hexadecimal notation
{% endhighlight %}

4\. 浮点指数形式

{% highlight swift %}
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1
let hexadecimalDouble = 0xC.3p0
{% endhighlight %}

5\. 字面量可度性

{% highlight swift %}
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
{% endhighlight %}

6\. 类型别名

{% highlight swift %}
typealias AudioSample = UInt16
{% endhighlight %}

7\. 元组聚合/分解/访问

{% highlight swift %}
let response = (statusCode: 200, description: "OK")
let (statusCode, _) = response
let statusCode = response.statusCode
let statusCode = response.0
{% endhighlight %}

8\. 可选类型

{% highlight swift %}
var string: String 	// 未初始化, 不可为nil.
var string: String?	// 已初始化, 可为nil, 方法调用时需unwrap.
var string: String!	// 已初始化, 可为nil, 方法调用时无需unwrap, 但若值为nil, 触发runtime error.
{% endhighlight %}

9\. 可选类型的本质:

	var optional: Type? 等同于 var optional: Optional<Type>

	var implicit: Type! 等同于 var implicit: ImplicitlyUnwrappedOptional<Type>

10\. 处理可选类型:

	

{% highlight swift %}
// a) 强制解析
if convertedNumber != nil {
    println("\(possibleNumber) has an integer value of \(convertedNumber!)")
} 

// b) 可选绑定
if let actualNumber = possibleNumber.toInt() {
    println("\(possibleNumber) has an integer value of \(actualNumber)")
}

// c) 隐式解析
let assumedString: String! = "An implicitly unwrapped optional string."
println(assumedString)

// 可选绑定陷阱, 可选绑定只判定optional.
let bool: Bool? = false
if let value = bool {	// value = false
    println("whatever")	// 正确的写法需要在这里判别value
}
{% endhighlight %}

11\. 奇葩的保留字使用方式, 使用数字1左侧的'`'包裹保留字, 访问时另需括号包裹.

{% highlight swift %}
var `class` = 5
(`class`) = 6
println(`class`)
{% endhighlight %}

## Basic Operators

1\. 返回值

	赋值操作(=), 复合赋值操作(+=/-=...)返回Void

2\. 空合运算符, 空合运算属于短路运算

	a ?? b 等同于 a != nil ? a! : b

3\. 区间

	1..<5 包含1、2、3、4; 1...5 包含1、2、3、4、5

## Strings and Characters

1\. String是值类型. var/let决定mutable与否.

2\. 插值语法

{% highlight swift %}
let multiplier = 3
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"
// message is "3 times 2.5 is 7.5"
{% endhighlight %}

3\. 字符计数与书写字簇(extended grapheme clusters)

{% highlight swift %}
var word = "cafe"
println("the number of characters in \(word) is \(countElements(word))")
// prints "the number of characters in cafe is 4"
 
word += "\u{301}"    // COMBINING ACUTE ACCENT, U+0301
println("the number of characters in \(word) is \(countElements(word))")
// prints "the number of characters in café is 4"
{% endhighlight %}

	Extended grapheme clusters can be composed of one or more Unicode scalars. This means that different characters, and different representations of the same character, can require different amounts of memory to store. Because of this, characters in Swift do not each take up the same amount of memory within a string’s representation. As a result, the number of characters in a string cannot be calculated without iterating through the string to determine its extended grapheme cluster boundaries. If you are working with particularly long string values, be aware that the countElements function must iterate over the Unicode scalars in the entire string in order to calculate an accurate character count for that string.

	扩展书写字符簇可以组成一个或多个Unicode标量. 这就意味着, 不同字符和相同字符的不同编码都可能导致所需存储空间的不同. 因此, Swift中字符占用的存储空间不等同于string编码所占用的. 于是, string中的字符数必须通过遍历以确定扩展书写字符的边界后才能计算得知. 在使用特别长的字符串时, 需注意countElements函数必须遍历整个string的Unicode标量才能计算出准确的字符数.

	Note also that the character count returned by countElements is not always the same as the length property of an NSString that contains the same characters. The length of an NSString is based on the number of 16-bit code units within the string’s UTF-16 representation and not the number of Unicode extended grapheme clusters within the string. To reflect this fact, the length property from NSString is called utf16Count when it is accessed on a Swift String value.

	也需注意, 相同字符下, countElements返回的字符数并不总等于NSString的length属性. NSString的length属性基于string的UTF－16编码下的16位编码单元数, 而不是Unicode扩展书写字符簇. 实际上, NSString的length属性就对应了Swift String的utf16Count.

4\. 编码

{% highlight swift %}
string.unicodeScalars 	// UnicodeScalarView类型, 21位数字, Swift原生String选用.
string.utf8			 	// UTF8View类型, UInt8集合.
string.utf16			// UTF16View类型, UInt16集合.
{% endhighlight %}

## Collection Types

1\. Array/Dictionary通过范型实现, 是值类型, var/let决定mutable与否. let Array/Dictionary的长度和内容皆不可变.

2\. 尽可能使用let, 编译器会执行优化操作.

3\. 数组追加

{% highlight swift %}
array.append("A")
array += ["A"]				// 即使只有一个元素也需要方括号
array += ["A", "B", "C"]
{% endhighlight %}

4\. 数组替换

{% highlight swift %}
array[0] = "A"
array[1...5] = ["B", "C"]	// 个数无需相等
{% endhighlight %}

5\. 数组枚举

{% highlight swift %}
for item in shoppingList {
    println(item)
}
for (index, value) in enumerate(shoppingList) {
    println("Item \(index + 1): \(value)")
}
{% endhighlight %}

6\. 支持多维数组

{% highlight swift %}
var array3D: [[[Int]]] = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]]
let element = array3D[1][0] // [5, 6]
{% endhighlight %}

7\. 字典替换

{% highlight swift %}
if let oldValue = dict.updateValue("VALUE", forKey: "KEY") {
    println("The old value for KEY was \(oldValue).")
}
{% endhighlight %}

8\. 字典枚举

{% highlight swift %}
for (key, value) in dict {
    println("\(key): \(value)")
}
{% endhighlight %}

## Control Flow

1\. 隐式break; 非隐式fallthrough.

{% highlight swift %}
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
{% endhighlight %}

2\. 区间匹配

{% highlight swift %}
switch count {
case 0:
    naturalCount = "no"
case 1, 2, 3, 4
    naturalCount = "few"
case 5...10:
    naturalCount = "some"
default:
    naturalCount = "lots of"
}
{% endhighlight %}

3\. 元组匹配

{% highlight swift %}
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    println("(0, 0) is at the origin")
case (let x, 0):
    println("(\(somePoint.0), 0) is on the x-axis")
case (0, let y):
    println("(0, \(y) is on the y-axis")
case (-2...2, -2...2):
    println("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    println("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
{% endhighlight %}

4\. 条件匹配

{% highlight swift %}
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    println("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    println("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    println("(\(x), \(y)) is just some arbitrary point")
}
{% endhighlight %}

5\. switch匹配时使用'~='操作符, [详参](http://andelf.github.io/blog/2014/06/17/nsobject-pattern-match-in-swift/?utm_source=tuicool).

{% highlight swift %}
let contains = (1...5 ~= 3)         // true
let contains = ("a"..."z" ~= "c")   // true
{% endhighlight %}

6\. 带标签的break

{% highlight swift %}
gameLoop: while square != finalSquare {
    if ++diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // 到达最后一个方块，游戏结束
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // 超出最后一个方块，再掷一次骰子
        continue gameLoop
    default:
        // 本次移动有效
        square += diceRoll
        square += board[square]
    }
}
{% endhighlight %}

7\. switch必须完备, default多数情况必须.

8\. case内容不能为空, 可以用break占位.

9\. for-in语句要求集合遵循SequenceType协议, 调用generate()生成enumerator, 而后通过GeneratorType协议的next()方法枚举.

## Functions

1\. 定义

{% highlight swift %}
func <Function Name>(<Parameters List>) -> <Return Type> {
	// function body
	return <Return Value>
}
{% endhighlight %}

Void返回类型的函数一样有空元组()作为返回值.

2\. 外部参数名

{% highlight swift %}
func function(p1: Int, _ p2: Int, #p3: Int, param4 p4: Int) {
    // whatever
}
function(1, 2, p3: 3, param4: 4)
{% endhighlight %}

3\. 默认值, 应该把带有默认值的参数置于参数列表的最后, 以保证参数顺序与使不使用默认值无关.

{% highlight swift %}
func join(s1: String, s2: String, joiner: String = " ") -> String {
    return s1 + joiner + s2
}
join("hello", "world", joiner: "-") // 默认值参数项会自动添加外部参数名(同#), 可以使用下划线(_)移除外部参数名
{% endhighlight %}

4\. 变长参数, 每个函数只能有一个变长参数, 且必须置于参数列表的最后, 且需置于默认值参数之后. 变长参数在函数内视为数组.

{% highlight swift %}
func arithmeticMean(numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
{% endhighlight %}

5\. 参数若非声明为var, 皆默认为let.

6\. In-Out参数

{% highlight swift %}
func swapTwoInts(inout a: Int, inout b: Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
swapTwoInts(&someInt, &anotherInt)	// 调用时需加&, 格式与传引用相似
{% endhighlight %}

7\. 函数/嵌套/柯里化

{% highlight swift %}
// function
func add(to: Int, value: Int) -> Int {
    return value + to
}
// nested function
func addNested(to: Int) -> (Int -> Int) {
    func advanceBy(value: Int) -> Int {
        return value + to;
    }
    return advanceBy
}
// curried function
func addCurried(to: Int)(value: Int) -> Int {
    return value + to
}

let result = add(1, 2)                      // 1 + 2 = 3
let nestedFunc = addNested(1)               // 1 + ? = ?
let nestedResult = nestedFunc(2)            // 1 + 2 = 3
let curriedFunc = addCurried(1)             // 1 + ? = ?
let curriedResult = curriedFunc(value: 2)   // 1 + 2 = 3
{% endhighlight %}

8\. 在函数使用有返回值但无数参的函数作为参数时, 可以使用autoclosure延迟表达式的计算, 自动包装表达式为无参闭包.

{% highlight swift %}
func simpleAssert(condition: @autoclosure () -> Bool, message: String) {
    if !condition() {
        println(message)
    }
}
simpleAssert(5 % 2 == 0, "5 isn't an even number.")
{% endhighlight %}

## Closures

1\. 闭包属于引用类型. 闭包视同函数. 但参数不支持默认值. 表达式如下:

{% highlight swift %}
let backwards: (String, String) -> Bool = {
    (s1: String, s2: String) -> Bool in
    return s1 > s2
}
let reversed = sorted(names, backwards)
{% endhighlight %}

2\. 残暴的类型推断/隐式返回/参数缩写

{% highlight swift %}
// 类型推断
let backwards: (String, String) -> Bool = {
    s1, s2 in
    return s1 > s2
}

// 隐式返回
let backwards: (String, String) -> Bool = {
    (s1: String, s2: String) -> Bool in
    s1 > s2
}

// 参数缩写
let backwards: (String, String) -> Bool = {
    return $0 > $1
}

// 类型推断+隐式返回+参数缩写
let backwards: (String, String) -> Bool = {
    $0 > $1
}
{% endhighlight %}

3\. 恐怖的运算符函数

{% highlight swift %}
let reversed = sorted(names, >)
{% endhighlight %}

4\. 尾随闭包, 闭包作为函数最后一个参数时, 可以以尾随形式书写闭包.

{% highlight swift %}
func someFunctionThatTakesAClosure(closure: () -> ()) {
    // function body goes here
}
// 普通闭包
someFunctionThatTakesAClosure({
    // closure's body goes here
})
// 尾随闭包
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
// 闭包作为函数唯一参数时括号都可以省略
someFunctionThatTakesAClosure {
    // trailing closure's body goes here
}
{% endhighlight %}

5\. 值捕获, 同Block, Swift会自动决定捕获引用还是拷贝值, 同时负责内存管理.

## Enumerations

1\. Swift创建枚举成员无需关联整数值

{% highlight swift %}
enum CompassPoint {
    case North	// 不等于0
    case South
    case East
    case West
}
{% endhighlight %}

2\. 单行多个case

{% highlight swift %}
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
{% endhighlight %}

3\. 类型推断

{% highlight swift %}
let directionToHead: Planet = .Venus
{% endhighlight %}

4\. 相关值, 语法近似元组聚合/分解

{% highlight swift %}
enum Barcode {
    case UPCA(Int, Int, Int, Int)
    case QRCode(String)
}
var productBarcode = Barcode.UPCA(8, 85909, 51226, 3)

switch productBarcode {
case .UPCA(let numberSystem, let manufacturer, let product, let check):
    println("UPC-A: \(numberSystem), \(manufacturer), \(product), \(check).")
case .QRCode(let productCode):
    println("QR code: \(productCode).")
}
{% endhighlight %}

5\. 原始值, 可以使用string/character/整型/浮点型值, 需保证值唯一. 当整型用于原始值时, 若未设定枚举成员值, 会自动递增.

{% highlight swift %}
enum ASCIIControlCharacter: Character {
    case Tab = "\t"
    case LineFeed = "\n"
    case CarriageReturn = "\r"
}
{% endhighlight %}

6\. 通过原始值初始化, 返回值类型为optional.

{% highlight swift %}
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet is of type Planet? and equals Planet.Uranus
{% endhighlight %}

## Classes and Structures

1\. class与struct的相同点:

	- 定义属性用于存储值
	- 定义方法用于提供功能
	- 定义下标脚本用于访问值
	- 定义构造器用于生成初始化值
	- 通过扩展以增加默认实现的功能
	- 符合协议以对某类提供标准功能

2\. class与struct的不同点:

	- class支持继承
	- class支持多态
	- class支持deinit
	- class实例是引用类型, 而struct实例是值类型

3\. 属性访问, Swift允许直接修改结构体属性的子属性

{% highlight swift %}
view.frame.size.width = 100
{% endhighlight %}

4\. 结构体构造器

	a) 默认构造器, 所有存储属性都有默认值时自动生成

	b) 成员初始化构造器, 未提供任何构造器时自动生成, 以初始化实例中所有的存储属性

5\. 恒等运算符. ===用于判别两个对象(限定class)是否同一实例, !==反之.

6\. Swift会管理所有值拷贝以优化性能, 无需避免赋值操作. 只有在确实必要时, Swift才会执行真正的拷贝. (Copy on Write ?)

## Properties

1\. Swift不区分property/instance variable, 统一为属性.

2\. 实例存储属性. 可为var/let; 仅class/struct支持.

3\. 实例计算属性, 限定为var; class/struct/enum都支持.

4\. 实例lazy属性, 初次访问时初始化. 限定为var. 

5\. 全局常量/变量总延迟计算, 但无需标记为lazy; 局部变量永远不会延迟计算.

6\. 类型属性. struct/enum用static标记, 支持存储/计算属性. class用class标记, 仅支持计算属性. 类型存储变量必须有默认值, 因为不存在类型构造器以初始化类型存储变量.

{% highlight swift %}
// 且算变通
class SomeClass {
    struct storedTypeProperty {
        static var value: Int = 0
    }
    class var computedTypeProperty: Int {
        get {
            return self.storedTypeProperty.value
        }
        set {
            self.storedTypeProperty.value = newValue
        }
    }
}
{% endhighlight %}

7\. 属性观察者willSet/didSet, 限用于a)非lazy存储属性 b)任意继承属性(存储/计算皆可).

8\. 属性观察者willSet/didSet在构造器委托前不会调用.

{% highlight swift %}
class Superclass {
    var value: Int! {
        didSet {
            self.value 		// console: 3
        }
    }
    init(value: Int) {
        self.value = value
    }
}
class Subclass: Superclass {
    var subvalue: Int! {
        didSet {
            self.subvalue	// not triggered at all
        }
    }
    override init(value: Int) {
        self.subvalue = 2
        super.init(value: value)
        self.value = 3     // trigger
        self.subvalue = 4
    }
}
var sub = Subclass(value: 1)
{% endhighlight %}

9\. didSet中可以覆盖属性值, 而不会再次触发观察者.

{% highlight swift %}
// 这是个反例...
class Superclass {
    var storedInt: Int = 0
    var calculatedInt: Int {
        get {
            return self.storedInt
        }
        set {
            self.storedInt = newValue		// called twice with 1, 0
        }
    }
}

class Subclass: Superclass {
    override var calculatedInt: Int {
        didSet {
            self.calculatedInt
            
            if self.calculatedInt > 0 {
                self.calculatedInt -= 1;	// called twice with 1, 0
            }
        }
    }
}

var instance = Subclass()
instance.calculatedInt = 1
{% endhighlight %}

10\. setter和willSet/didSet无法共存于单个类型层次中.

## Methods

1\. Swift默认给除首参之外的后续参数添加外部参数名, 效果同(#).

2\. 结构体和枚举是值类型, 如需在方法哪修改属性, 需要在func前增加mutating标识. let结构体/枚举变量不能调用mutating方法.

3\. mutating方法可以self赋值.

{% highlight swift %}
struct Point {
  var x = 0.0, y = 0.0
  mutating func moveByX(deltaX: Double, y deltaY: Double) {
    self = Point(x: x + deltaX, y: y + deltaY)
  }
}
{% endhighlight %}

4\. 类型方法. struct/enum用static标记. class用class标记.

5\. 支持overload.

## Subscripts

1\. 下标脚本支持overload, 语法与属性近似, 但需至少一个参数.

{% highlight swift %}
subscript(index: Int) -> Int {
    get {
        // return an appropriate subscript value here
    }
    set {
        // perform a suitable setting action here
    }
}
{% endhighlight %}

2\. 下标脚本参数不能使用inout标识, 也不能给定默认值.

## Inheritance

1\. Swift类没有通用基类.

2\. 子类可以override实例方法/类方法/实例属性/类属性和下标脚本.

3\. 子类可以override属性的getter和setter, 无论存储/计算属性. 若需override setter, 必须同时提供getter.

4\. 子类可以override属性的willSet和didSet, 限用于非常量存储属性和非只读计算属性.

5\. 使用final阻止实例方法/类方法/实例属性/类属性和下标脚本被override, 或阻止类被继承.

## Initialization

1\. class和struct必需在创建实例时, 为所有的存储属性提供初值.

2\. 设定存储属性默认值不会触发观察者.

3\. 使用默认值设定初值的方式除了类型推断/简化构造器的好处外, 还利于默认构造器和构造器继承.

4\. 可选属性类型的默认值为nil.

5\. 不同于方法, Swift默认为构造器的首参也提供了外部参数名.

6\. 在构造器中, 可以修改常量存储属性, 但子类不能修改父类的常量存储属性.

7\. Swift为提供了所有存储属性默认值, 但没有提供构造器的struct和base class提供默认构造器.

8\. 推荐使用委托构造器简化构造器逻辑; 如需保留默认构造器和成员构造器, 可以在extension中定义委托构造器.

9\. class有两类构造器, 构造过程分为两个阶段
    
{% highlight swift %}
// a) designated initializer, 每个类至少有一个, 需初始化所有class引入的存储属性, 并调用父类的构造器以完成链式构造. 
init(...) {
    // first phase - initialize ALL stored property
    self.storedProperty = ...   // if default value is not provided
    super.init(...)             // delegate up

    // second phase - customize stored property
    self.storedProperty = ...
    super.storedProperty = ...
}

// b) convenience initializer, 数量不限, 处理输入并调用class的designated initializer以委托构造.
convenience init(parameters) {
    // first phase - initialize ALL stored property
    self.init(...)              // delegate across, should be a designated initializer

    // second phase - customize stored property
    self.storedProperty = ...
    super.storedProperty = ...
}
{% endhighlight %}

    first phase
    a) 调用designated initializer或convenience initializer
    b) 为新实例分配内存, 此时内存未初始化
    c) 初始化类引入的所有存储属性
    d) 逐层调用父类构造器, 完成后first phase完成, 所有存储属性都已初始化

    second phase
    a) 自定向下定制实例. 构造器职能在second phase中调用实例方法/读取实例属性/使用self.

10\. override designated initializer时, 必需添加override标志, 即使改写为convenience initializer.

11\. override convenience initializer时, 由于子类无法直接调用父类的convenience initializer, 不构成严格意义上的override, 因此不能使用override标志.

12\. 构造器继承规则:

    a) 当子类未实现任何designated initializer时, 继承父类所有的designated initializer.

    b) 当子类实现/继承了父类的所有designated initializer时(可以改写为convenience initializer), 继承父类所有的convenience initializer.

    c) 除此之外, Swift默认不继承构造器以免子类不完整构造.

13\. 可以使用init?或init!标识class/struct/enum构造器可能失败, init?与init!及init不构成overload. 

14\. struct/enum可以在构造器的任意位置返回nil以表明构造失败, 但class必须在初始化class引入的存储属性并委托构造后才能返回nil.

15\. 如果构造器委托构造给可能失败的构造器, 那么构造器本身也需要标识为可能失败. 可能失败的构造器可以委托给不失败的构造器, 但不失败的构造器不能委托给可能失败的构造器.

16\. 可以将可能失败的构造器override为不失败的构造器, 但不能将不失败的构造器override为可能失败的构造器.

17\. 可以使用required标识出子类必需实现/继承的构造器. 若子类继承required构造器, 则required失效, 子类的子类无需实现; 若子类实现required构造器, 仍需标记为required, 而非override.

{% highlight swift %}
class Animal {
    required init () {

    }
}
class Mammal: Animal {

}
class Human: Mammal {   // 不必实现init, 危险的required丢失特性
    var name: String
    init(name: String) {
        self.name = name;
        super.init()
    }
}
var human = Human(name: "")
{% endhighlight %}

18\. 可以使用函数或闭包提供存储属性的初值. 但需特别注意的是, 初值设定属于构造的first phase, 闭包中不应调用实例方法/读取实例属性/显式或隐式的使用self.

{% highlight swift %}
class SomeClass {
    let someProperty: SomeType = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
{% endhighlight %}

## Deinitialization

1\. 析构器仅class有. 由无参方法deinit定义. 父类的析构器在子类析构器完成后自动调用.

## Automatic Reference Counting

1\. 引用计数限用于引用类型, 即class.

2\. 属性引用默认为强引用. 使用weak标识可选的弱引用, 弱引用限用于变量; 使用unowned标识非可选或隐式解析的非持有引用, 非持有引用对象销毁后若访问属性, 则触发运行时错误.

3\. 结合unowned和隐式解析可以解决初始化过程中无法self的问题.

{% highlight swift %}
class Country {
    let name: String
    let capitalCity: City!          // 隐式解析, 默认值nil
    init(name: String, capitalName: String) {
        self.name = name            // first phase完成
        self.capitalCity = City(name: capitalName, country: self)   // 可以在capitalCity没有actual值的情况下使用self
    }
}
class City {
    let name: String
    unowned let country: Country    // 解除循环引用
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
{% endhighlight %}

4\. 闭包捕获引用默认为强引用. 使用捕获表改变引用性质, 如weak和unowned. Swift要求在闭包内使用成员时, 必需使用self, 以免遗忘把self加入捕获列表.

{% highlight swift %}
lazy var someClosure: () -> String = {
    [unowned self, weak someInstance = self.somewhat] in
    // closure body goes here
}
{% endhighlight %}

## Optional Chaining

1\. 可选链与Objective-C中的nil相似, 但额外提供检测成功失败的功能.

{% highlight swift %}
class Person {
    var residence: Residence?
}
class Residence {
    var numberOfRooms = 1

    func printNumberOfRooms() {
        println("The number of rooms is \(numberOfRooms)")
    }
}

// 由于存在可选链, 即使numberOfRooms的推断类型为Int, number的类型也会是Int?. 
// 可选性质不受链内可选节点数影响. 
// 链内任意节点为nil都会导致整条链失败. 
let number = consumers.first?.residence?.numberOfRooms

// 当方法有返回值时, 处理方式与访问属性相同. 
// 当方法返回类型为Void时, 其本质是空元组(), 则可选链的返回类型为()?.
// 可以通过返回值判定调用是否成功.
let succ: ()? = consumers.first?.residence?.printNumberOfRooms()
if succ != nil {
	println("It was possible to print the number of rooms.")
}

// 赋值语句类似于方法调用
if (consumers.first?.residence?.numberOfRooms = 3) == nil {
    println("Failed in setting the number of rooms.")
}
{% endhighlight %}

2\. 下标脚本也可以是可选链的一部分, 但需特别留意'?'需要置于'[]'之前.

{% highlight swift %}
var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
testScores["Dave"]?[0] = 91		// set
testScores["Bev"]?[0]++			// get
{% endhighlight %}

## Type Casting

1\. 使用'is'做类型检查

{% highlight swift %}
if item is Movie {
    ++movieCount
} 
{% endhighlight %}

2\. 使用'as'编译期安全转型, 如向上转型或bridge; 'as?'可选转型, 转型失败时返回nil; 'as!'强制转型, 失败时触发运行时错误. x as! T 等同于 (x as? T)!.

{% highlight swift %}
// as! example
for movie in library as! [Movie] {
    println("Movie: '\(movie.name)', dir. \(movie.director)")
}

// as? example
for object in someObjects {
    if let movie = object as? Movie {
        println("Movie: '\(movie.name)', dir. \(movie.director)")
    }
}

// 在switch中使用as, 而非as?, 以检查类型. 
for thing in things {
    switch thing {
    // 0值判别
    case 0 as Int:
        println("zero as an Int")
    case 0 as Double:
        println("zero as a Double")

    // Int 判别
    case let someInt as Int:
        println("an integer value of \(someInt)")
    // Double 正数判别
    case let someDouble as Double where someDouble > 0:
        println("a positive double value of \(someDouble)")
    // Double 负数判别
    case is Double:
        println("some other double value that I don't want to print")

    // String 判别
    case let someString as String:
        println("a string value of \"\(someString)\"")
    // 元组值绑定
    case let (x, y) as (Double, Double):
        println("an (x, y) point at \(x), \(y)")
    // func 判别
    case let stringConverter as String -> String:
        println(stringConverter("Michael"))
    default:
        println("something else")
    }
}
{% endhighlight %}

3\. 可以使用AnyObject表示任意class实例; 可以使用Any表示任意class/struct/enum/func实例.

## Nested Types

1\. Swift允许在class/struct/enum定义中嵌套类型, 嵌套层级不限.

{% highlight swift %}
struct Card {
    enum Suit: Character {
       case Spades = "♠", Hearts = "♡", Diamonds = "♢", Clubs = "♣"
    }
    enum Rank: Int {
       case Two = 2, Three, Four, Five, Six, Seven, Eight, Nine, Ten
       case Jack, Queen, King, Ace
    }
}

enum Rank: Int {
   case Two = 2, Three, Four, Five, Six, Seven, Eight, Nine, Ten
   case Jack, Queen, King, Ace
}

let r1 = Rank.Two
let r2 = Card.Rank.Two
let r3 = Card.Rank.Two

if r1 == r2 {           // compile error
    println("equals")
}
if r2 == r3 {           
    println("equals")   // prints equal
}
{% endhighlight %}

## Extensions

1\. Extension可以,

    a) 添加实例/类型计算属性, 但不能添加存储属性, 也不能添加属性观察者.
    b) 添加实例/类型方法, 可以添加mutating方法, 但不能override方法.
    c) 添加新的convenience构造器, 但不能添加designated构造器, 在extension中添加构造器不会影响默认构造器和成员构造器的自动生成.
    d) 添加新的下标脚本.
    e) 添加新的嵌套类型, 不限于class/struct/enum.
    f) 使类型符合指定协议.

## Protocols

1\. Protocol可以, 

    a) 要求类型提供实例/类型属性, 及读写特性(get/set), 但不能限定属性的实现方式(存储/计算). 类型属性无论class/struct/enum, 一律用class标识.

{% highlight swift %}
protocol AnotherProtocol {
    class var someTypeProperty: Int { get set }
}
{% endhighlight %}

    b) 要求类型提供实例/类型方法, 可以有变长参数, 但参数不能设定默认值. 类型方法无论class/struct/enum, 一律用class标识.
    c) 要求类型提供mutating的实例方法, struct/enum在实现方法时需添加mutating标记, 但class不受此限制.
    d) 要求类型提供指定构造器, 但不限制实现方式(convenience/designated), 除final class外, class实现protocol构造器必需添加required标识. 协议构造器可以运行构造失败, 可失败的协议构造器可以通过可失败/不可失败构造器满足; 不可失败的协议构造器可以通过不可失败/隐式解析构造器满足.

{% highlight swift %}
protocol AgeRequired {
    init(age: UInt)
}
class Animal: AgeRequired {
    required init (age: UInt) { // required必不可少
        // whatever
    }
}
class Mammal: Animal {
}
class Human: Mammal {
    var name: String
    init(name: String) {
        self.name = name;
        super.init(age: 0)
    }
}
var human = Human(name: "") // init(age:)特性丢失, init默认不继承特性也是挺危险的.
{% endhighlight %}

    e) 要求类型提供operator或下标脚本.

2\. 满足协议并不等同于conform, 必需显示声明, 当已有实现已经满足协议时, 可以使用extension声明, 而{}中留空即可.
3\. 协议可以继承, 且可以多重继承.
4\. 协议可以通过在继承列表的首位加入class, 以声明协议仅限用于class, 这在协议要求实现者具有引用特性而非值特性时尤为有用.

{% highlight swift %}
protocol SomeClassOnlyProtocol: class, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
{% endhighlight %}

5\. 可以通过protocol<protocol1, protocol2>的方式定义一个临时的局部合成协议.
6\. protocol一样使用'is'做类型检查, 用'as'做向上转型或bridge, 用'as?'或'as!'做向下转型.
7\. 可以在protocol中使用optional标识方法或属性可以选实现, 但protocol必需使用@objc标识为面向Objective-C, 且一旦标记为@objc, protocol就限用于class.

{% highlight swift %}
@objc protocol CounterDataSource {
    optional func incrementForCount(count: Int) -> Int
    optional var fixedIncrement: Int { get }
}
{% endhighlight %}

## Generics

1\. 范型方法

{% highlight swift %}
func swapTwoValues<T>(inout a: T, inout b: T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3, anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
{% endhighlight %}

2\. 范型类型

{% highlight swift %}
struct Stack<T> {
    var items = [T]()
    mutating func push(item: T) {
        items.append(item)
    }
    mutating func pop() -> T {
        return items.removeLast()
    }
}
{% endhighlight %}

3\. 拓展范型类型时, 无需提供类型参数列表.

{% highlight swift %}
extension Stack {
    var topItem: T? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
{% endhighlight %}

4\. 类型约束

{% highlight swift %}
func allItemsMatch <
    C1: Container, 
    C2: Container where C1.ItemType == C2.ItemType, 
    C1.ItemType: Equatable
> (someContainer: C1, anotherContainer: C2) -> Bool {
    // check that both containers contain the same number of items
    if someContainer.count != anotherContainer.count {
        return false
    }  
    // check each pair of items to see if they are equivalent
    for i in 0..<someContainer.count {
        if someContainer[i] != anotherContainer[i] {
            return false
        }
    }
    // all items match, so return true
    return true
}
{% endhighlight %}

5\. 协议与关联类型

{% highlight swift %}
protocol Container {
    typealias ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}

struct Stack<T>: Container {
    // original Stack<T> implementation
    var items = [T]()
    mutating func push(item: T) {
        items.append(item)
    }
    mutating func pop() -> T {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias ItemType = T
    mutating func append(item: T) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> T {
        return items[i]
    }
}
{% endhighlight %}

    其中的'typealias ItemType = T'无需显式申明, 在实现满足协议时, 可以自动推断.
    但若存在多种ItemType的overload时呢...? typealias语法看起来会非常不直观. 范型协议迟早还是要支持的比较好.

## Access Control

1\. 可以为 a) 类型(class/struct/enum)及其属性、方法、构造器、下标脚本 b) 协议 c) 全局常量/变量和函数 设定访问级别。

2\. module, 独立代码发布单元, framework或application, 在另一个module中通过import关键字引入; source file, module中的单一代码文件.

3\. 三种访问级别:

    a) public, 最高级别, 任意访问, 多用于framework接口;
    b) internal, 默认级别, module内访问, 多用于application/framework内部逻辑;
    c) private, 最低级别, source file内访问, 隐藏实现.

4\. 访问级别总原则: 类型访问级别递减, 保障限制; 成员访问级别递增, 保障可见性.

    a) 枚举类型. 无法单独为case指定访问级别, 统一使用enum的访问级别; 若使用raw value或相关值, 其类型访问级别不得低于enum本身.
    b) 元组类型. 不同与class/struct/enum, 它没有独立的定义, 无法显式声明访问级别, 因此访问级别自动推断, 以最低的内容类型访问级别为准.
    c) 函数类型. 以参数和返回值类型的最低访问级别作为推断级别, 不显式指定访问级别时使用推断级别, 指定级别不得低于推断级别.

    d) 若类型访问级别为public或internal, 属性、方法、构造器、下标脚本的默认访问级别为internal;
       若类型访问级别为private, 属性、方法、构造器、下标脚本的默认访问级别为private.
    e) 若类型访问级别为public或internal, 则其嵌套类型的默认访问级别为internal; 
       若类型访问级别为private, 则其嵌套类型的默认访问级别为private.
    f) 子类的访问级别不能高于父类, 但override父类成员时可以使用更高的访问级别.

    g) 自定义构造器的访问级别不高于类型访问级别; required构造器访问级别必需等同于类型;
    h) 若类型访问级别为public或internal, 默认构造器访问级别为internal;
       若类型访问级别为private, 默认构造器访问级为private;
       如需public默认构造器, 需显式声明.
    i) 若struct无private存储属性, 则默认成员构造器访问级别为internal;
       若struct有private存储属性, 则默认成员构造器访问级别为private; 
       如需public默认成员构造器, 需显式声明.

    j) 常量/变量/属性/下标脚本的访问级别不得其高于类型, 下标基本访问级别也不得高于索引类型和返回值类型. 
       setter/getter访问级别默认等同于常量/变量/属性/下标脚本的访问级别; setter的访问级别可以低于getter.
       eg. public private(set) var numberOfEdits = 0

    k) 协议内容的访问级别默认等同于协议访问级别, 协议内容的访问级别不得低于协议访问级别, 以保证协议内容的可见性.
    l) 子协议访问级别不能高于父协议.
    m) 类型实现协议时, 可以使用低于自身的访问级别, 如public type + internal protocol.

    n) 若类型访问级别为public或internal, extension的默认访问级别为internal;
       若类型访问级别为private, extension的默认访问级别为private;
       可以显式指定extension的访问级别.
    o) 使用extension添加协议支持时, 不能显式指定访问级别, 必需使用协议定义的访问级别.

    p) 范型类型和范型函数的访问级别限制需符合所使用类型及其成员的所有访问级别限制.
    q) 类型别名视为不同类型, 别名类型的访问级别不得高于源类型.

## Advanced Operators

1\. 位移. 无符号整型左右位移抛弃超出部分, 空档填0; 带符号整型左移可溢出, 右移以符号位填空档.

{% highlight swift %}
var pi: Int8 = 2    // 0000 0010
var ni: Int8 = -2   // 1111 1110
var ui: UInt8 = 2   // 0000 0010

pi << 5 // 0100 0000 = 64
ni << 5 // 1100 0000 = -64
ui << 5 // 0100 0000 = 64

pi << 6 // 1000 0000 = -128 [overflow]
ni << 6 // 1000 0000 = -128
ui << 6 // 1000 0000 = 128

pi << 7 // 0000 0000 = 0
ni << 7 // 0000 0000 = 0
ui << 7 // 0000 0000 = 0

pi >> 1 // 0000 0001 = 1
ni >> 1 // 1111 1111 = -1
ui >> 1 // 0000 0001 = 1

pi >> 2 // 0000 0000 = 0
ni >> 2 // 1111 1111 = -1
ui >> 2 // 0000 0000 = 0
{% endhighlight %}

2\. 算数操作符. 与C不同, Swift的默认算数操作符(+/-/*)不溢出, 而是触发运行时错误; 如果需要启用溢出行为, 需使用&+/&-/&*操作符.

{% highlight swift %}
var overflow = UInt8.max &+ 1   // 0, 上溢, 从最大到最小
var underflow = Int8.min &- 1   // 127, 下溢, 从最小到最大
{% endhighlight %}

3\. 在global级别使用operator关键字和prefix/infix/postfix标识符声明操作符.

    <prefix/infix/postfix> operator <operator_token> {associativity <left/right/none> precedence <precedence_value>}

    <prefix/infix/postfix>
    prefix标识前缀操作符, infix标识中缀操作符, postfix标识后缀操作符.

    <operator_token>
    不能自定义的操作符: (, ), {, }, [, ], ., ,, :, ;, =, @, #, &(仅前缀), ->, `, ?, !(仅后缀), //, /*, */, <(仅前缀), >(仅中后缀).
    自定义操作符可以以/, =, -, +, !, *, %, <, >, &, |, ^, ?, ~, 或合规的Unicode(组合Unicode字符亦可)为开始.

    <left/right/none> 
    结合性, 默认为none.
    左结合操作符邻接同优先级的左结合操作符时左结合;
    右结合操作符邻接同优先级的右结合操作符时右结合;
    非结合操作符无法邻接同优先级的操作符.

    eg.
    {% highlight swift %}
infix operator +~+ {associativity left precedence 140}
func +~+ (lhs: Int, rhs: Int) -> Int {
    println("\(lhs) + \(rhs)")
    return lhs + rhs
}
let sum = 1 +~+ 2 +~+ 3 // 1 + 2, 3 + 3

infix operator +~+ {associativity right precedence 140}
func +~+ (lhs: Int, rhs: Int) -> Int {
    println("\(lhs) + \(rhs)")
    return lhs + rhs
}
let sum = 1 +~+ 2 +~+ 3 // 2 + 3, 1 + 5

infix operator +~+ {associativity none precedence 140}
func +~+ (lhs: Int, rhs: Int) -> Int {
    println("\(lhs) + \(rhs)")
    return lhs + rhs
}
let sum = 1 +~+ 2 +~+ 3 // compile error
{% endhighlight %}

    <precedence_value>
    优先级, 默认100, 同三元操作符'?:', 仅高于赋值操作符.
    定义prefix/postfix操作符时不指定优先级, 如果同时使用前缀后缀操作符, 后缀总是先执行.

4\. Swift中有四种expression, evaluating an expression returns a value, causes a side effect, or both.

    a) prefix expressions

    前缀表达式包含一个可选前缀操作符和一个表达式, 前缀操作符以后续表达式为参数. 
    标准库包含下列前缀操作符: 
    ++, --  增, 减
    !, ~    逻辑否, 位反
    +, -    正负号
    &       传引用

    b) binary expressions

    二元表达式包含一个中缀二元操作符和前后两个作为操作符参数的表达式.
    标准库包含下列二元操作符: 
    <<, >>                              左右位移, (No associativity, precedence level 160)
    *, /, %, &*, &                      乘, 除, 余, 溢出乘, 位与, (Left associative, precedence level 150)
    +, -, &+, &-, |, ^                  加, 减, 溢出加, 溢出减, 位或, 异或, (Left associative, precedence level 140)
    ..<, ...                            段, 包含段, (No associativity, precedence level 135)
    is, as, as?, as!                    类型检查, 安全转型, 可选转型, 强制转型, (No associativity, precedence level 132)
    ??                                  空合, (Right associative, precedence level 131)
    <, <=, >, >=, ==, !=, ===, !==, ~=  小, 小等, 大, 大等, 相等, 不等, 相同, 不同, 模式匹配, (No associativity, precedence level 130)
    &&                                  逻辑与, (Left associative, precedence level 120)
    ||                                  逻辑或, (Left associative, precedence level 110)
    ?:                                  三元条件, (Right associative, precedence level 100)
    =, *=, /=, %=, +=, -=, <<=, >>=,    等, 乘等, 除等, 余等, 加等, 减等, 左移等, 右移等, (Right associative, precedence level 90)
    &=, ^=, |=, &&=, ||=                位与等, 异或等, 或等, 逻辑与等, 逻辑或等, 

    c) primary expressions

    主表达式是最基础类型的表达式, 可以单独作为表达式, 也可以与其他操作符合用以生成前缀/二元/后缀表达式. 
    主表达式包含以下几类:
    字面量, 如string/number/array/dict;
    特殊字面量, __FILE__(String), __LINE__(Int), __COLUMN__(Int), __FUNCTION__(String);
    self表达式, self/self.member/self[index]/self(init_param)/self.init(init_param);
    super表达式, super.member/super[index]/super.init(init_param);
    closure表达式, {(parameters) -> return_type in statements};
    隐式成员表达式, var e: EnumType = .SomeValue;
    元组表达式, (identifier 1: expression 1, identifier 2: expression 2, ...);
    通配表达式, (x, _) = (10, 20).

    __FUNCTION__在函数中是函数名, 在方法中是方法名, 在属性定义中是属性名, 在init或subscript中为init或subscript, 在文件顶层是当前模块.

    d) postfix expressions

    后缀表达式由表达式和后缀操作符或其他后缀语法构成. 句法上, 每个primary expression也都是后缀表达式.
    标准库包含下列后缀操作符: 
    ++, --  增, 减

    后缀表达式包含以下几类:
    函数调用表达式, function name(name1: value1, name2: value2);
    构造器表达式, super.init();
    显式成员表达式, let y = c.someProperty;
    后缀self表达式, expression.self, type.self;
    动态类型表达式, expression.dynamicType;
    下标脚本表达式, expression[index];
    强制解析表达式, expression!;
    可选链表达式, expression?.

4\. 

	操作符前的(, [, {和操作符后的), ], }也视为空格.

	如果操作符两侧都有或都没有空格, 视为二元操作符;
	如果操作符仅左侧有空格, 视为一元前缀操作符;
	如果操作符仅右侧有空格, 视为一元后缀操作符;
	如果操作符左侧无空格, 但紧接点(.), 视为一元后缀操作符, eg, num++.description视同(num++).description.

	但有一个例外, 如果!或?预定义操作符左侧无空格, 无论右侧是否有空格, 都视为后缀操作符. 
	因此, 为了将'?'用于可选链, 左侧必需无空格. 而要使用'?:', 两侧必需有空格.

	在某些结构中, 以<或>打头的操作符可能会被拆分为多个符号, 余下的部分可能会再次被拆分, 因而无序使用空格来消除匹配字符>可能导致的歧义. 如Dictionary<String, Array<Int>>.

5\. 不可以overload =, ->, //, /*, */, ., <(仅前缀), &, ?, >(仅后缀), !和三元条件操作符(a ? b : c). 可以overload复合赋值操作符(如+=).

6\. 不会为自定义class和struct提供默认的判等操作符(==/!=), 相等逻辑需要由类型决定.
