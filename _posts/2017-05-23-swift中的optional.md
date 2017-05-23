# swift中的optional


## 什么是可选值
可选值是一种类型，就跟Int,String等基本类型一样。其底层实现如下,使用的了枚举中的关联值特性，获取值的时候需要解包才能使用，下面会介绍swift 提供的解包的方式
<pre>
enum Optional<Wrapped> {
    case none
    case some(wrapped)
}
</pre>

## swift 中的可选值是为了解决什么问题
swift 语言中设计可选类型主要是为了解决因为忘记检查nil而引起的Bug。swift 中的变量，属性在初始化的时候必须要有初始值，如果存在nil的情况就必须声明为optional类型，不管是基本值类型 还是引用类型。如果使用了声明为optional类型的变量而没有进行解包操作，编译器会给出相应的错误提醒。所以optional 机制强制程序员在定义变量的时候就必须考虑nil的情况，是否需要定义成optional 类型。如果定义成了optional类型，使用的时候编译器会强制考虑nil的情况，避免bug。

## 常见操作

### if let,if var

optional 底层实现是枚举，所以我们可以使用常用 while 来进行解包，但是除此以外 swift 还提供了if let ,if var 这两种方式进行解包操作<br>
**if let**<br>
使用if let进行解包的时候，后面还可以加上一个判断条件,如下，当索引不是数组的第一个索引的时候才进行删除。if let 也可以同时解绑多个可选值，后面的值可以使用前面解包的值
<pre>
var array = ["one", "two", "three", "four"]
if let idx = array.index(of: "four"),idx != array.startIndex {
    array.remove(at: idx)
}
</pre>
 **if var**<br>
 使用if var 的时候唯一不同的就是解包后的值是可以改变的，如optional 关联的是值类型，那么解包出来的值是一份拷贝，解包之后的值操作对以前的值不会产生影响,但是如果是引用类型，解包出来的是相同的引用，会对以前的值产生影响。
<pre>
var demo :String?
demo = "12312"
if var str = demo {
    str += "123123"
}
print(demo)//optioanl("123123")
</pre>

### while let，while var
**while let**<br>
while let 语句和 if let 非常相似，它代表一个当遇到 nil 时终止的循环。标准库中的 readLine 函数从标准输入中读取一个可选字符串。当到达输入末尾时，这个方法将返回 nil。所以，你可以使用 while let 来实现一个非常基础的和 Unix 中 cat 命令等价的函数。和 if let 一样，你可以在可选绑定后面添加一个布尔值语句。如果你想在遇到 EOF 或者空行的时候终止循环的话，只需要加一个判断空字符串的语句就行了。while let 在实际编码中使用较少，到目前为止我还没使用过。
<pre>
while let line = readLine(), !line.isEmpty {
print(line)
} 
</pre>

**while var**<br>
跟if var 类似，别什么特别的地方。

## 问题探索
### 可选值嵌套
optional 的value 还可以是 optional 类型的值，这样一层套一层，可以无限嵌套下去，但是实际中的确比较少遇到这样的使用场景。简单举一个栗子如下：
<pre>
var optional :String???
optional = "huang"
if let optional = optional,let optional1 = optional,let optional2 = optional1 {
    print(optional2)//"huang"
    print(optional1)//optional("huang")
    print(optional)//optional(optional("huang"))
}
</pre>

### 可选链(optional chaining)
可选链是在这次swift实践中遇到的一个比较突出的，把一些相关的 class 的属性定义为optional类型的，这些相关的class 之间互相嵌套的比较深，当要使用某个属性的时候，除了点操作以外还要写一连串的 ? ! ,代码显的不是非常美观，而连续解包代码也显得比较繁琐,但是相比较之下还是得用连续解包的方式，这样可以保证出现nil 的时候不会crash
<pre>
class Teachaer{
    var name = "teachaer"
    var student :Student?
}
class Student{
    var name = "student"
    var person :Person?
}
class Person{
    var name = "person"
}
var t1 = Teachaer()
var s1 = Student()
var p1 = Person()
t1.student = s1
s1.person = p1
var pname = (t1.student?.person?.name)!
print(pname)
if let student = t1.student,let person = student.person {
    print(person.name)
}
</pre>
