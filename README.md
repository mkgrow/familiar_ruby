# familiar_ruby
<br></br>
### 一、提防Ruby最棘手的解析
### 二、推荐使用Struct而非Hash存储结构化数据
### 三、通过在模块中嵌入代码来创建命名空间
<br></br>
### 一、提防Ruby最棘手的解析
#### Ruby 允许我们在方法末尾使用三个字母数字以外的符号：“ ?”、“ !”和“ =”
a、“？”被用于标示谓词，即返回Boolean直的方法，如Array.empty?(判断数组中元素是否为空)
<br>b、“！”感叹号常常意味着方法会改变接收者，也可以是对你的 示警，表示存在潜在的有害行为。如果出现在方法名尾部的那么使用该方法时需要多加小心。许多Ruby的核心类都定义了成对的方法，它们具有同样的名称，只是结尾相差一个“！”,通常情况下，不带感叹号的方法返调用该方法的一个拷贝，二带感叹号的方法则是一个可变方法，该方法会修改原来的对象，如Array类中的sort和sort!</br>
c、“ =”结尾的方法将变为 setter 方法

例如：
```
class SetValue
  def initialize
    @value = 0
  end 

  def value
    @value
  end

  def value= (x)
    @value = x
  end
end

x = SetValue.new
puts x.value = 1  (看似是变量赋值，但实质上仅仅是一个普通的方法调用。)
```
从技术上讲“ =”是方法名的一部分，Ruby 却允许我们在等号和方法名的其他部分之间输入空格
<br>puts x.value=(1)</br>
你可能不曾亲手定义过这些 setter 方法，但你一定间接地定义过。因为 Ruby 有一 些帮助方法能帮我们做这个事情。attr_writer 和 attr_accessor都像之前的例子一样定义了类似 value= 这样的 setter 方法。
<br>这样的迂回可能会让你有点困惑，这也是我提起它们 的原因。只要你记住这些帮助方法做的事情以及接下来的建议，
任何困难都不是问题。
<br>说到困难，我们得看看赋值和 setter 之间的区别。由于 setter 方法的调用看似变量 赋值，因此很容易混淆它们</br>
```
class Counter
  attr_accessor(:counter)
    def initialize
  counter = 0
  end
end
```
<br>把 initialize 方法体中的内容当作对 counter= 方法的调用也不是毫无道理，这也是 很多人所假设的。但这当然是不对的，那只是个简单的变量赋值呀。initialize 方法创建 了一个新的局部变量 counter，并将其赋值为 0，然后又在作用域结束的时候丢弃了对这 个变量的引用。当你仔细思考，会发现这显然不是我们想做的。
<br>Ruby 在对变量赋值和对 setter 方法调用时的解析是有区别的。区别在于，Ruby 调 用 setter 方法时要求存在一个显式接收者。为了调用 setter 方法，而非创建一个变量， 你需要预先指定方法名的接收者。因此，在实例方法中调用方法 counter=，你需要使用 self 充当这个接收者。</br>
```
class Counter
  attr_accessor(:counter)
  def  initialize
    self.counter = 0
  end
end
```
<br>通过使用 self 作为接收者，Ruby 将正确地解析你的代码并调用 counter= 这个 setter 方法，而非创建一个新的变量.
<br>避免分使用不必要的 self 接收者，这会使代码显得凌乱。</br>
```
例如：
class Name
  attr_accessor(:first, :last)
  def initialize
    self.first = first
    self.last = last
  end
  def full
    self.first + "" + self.last
  end
end
```

### 总结：    
##### setter 方法在调用时需要显式的接收者。没有接收者时，会被 Ruby 解析为变量赋值。
##### 在实例方法中调用 setter 方法时，使用 self 作为接收者。
##### 在调用非 setter 方法时，不需要显式指定接收者。换句话说，不要使用不必要的 self，那会弄乱你的代码。


### 二、推荐使用Struct而非Hash存储结构化数据
<br>哈希表是 Ruby 程序员经常使用的一种有用的、通用的数据结构。
<br>所有的哈希对象都包含相 同的键，不过值是不同的。本质上讲，这个哈希数组表示的是一些对象的集合，不能通过getter方法访问其属性，要访问其属性，你得通过哈希索引操作符来完成。
```
class  AnnualWeather
  def initialize
    @readings = [{:high => 2, :low => 1}, {:high => 2, :low => 1}, {:high => 2, :low => 1}, {:high => 2, :low => 1}]
  end
  def average
    return 0.0 if @readings.size.zero?
    total = @readings.reduce(0.0) do |sum, reading|
      sum + (reading[:high] + reading[:low]) / 2.0
    end
    total / (@reading.size.to_f)
  end
end
x = AnnualWeather.new
puts x.average 
```
<br>使用 Struct 非常简单，只需 调用 Struct::new 方法并附上属性列表。该方法的返回值几乎就是一个新类，它包含 每个属性的 getter 和 setter 方法。将 Struct::new 方法的返回值赋给一个常量是常见的实践。这允许你像 类一样使用这个常量，并利用它创建对象。</br>
```
class  AnnualWeather
  Reading = Struct.new(:high, :low)
  def initialize
    @readings = [Reading.new(2,1), Reading.new(2,1), Reading.new(2,1), Reading.new(2,1)]
  end
  def average
    return 0.0 if @readings.size.zero?
    total = @readings.reduce(0.0) do |sum, reading|
      sum + (reading.high + reading.low) / 2.0
    end
    total / (@reading.size.to_f)
  end
end
x = AnnualWeather.new
puts x.average 
```
<br>通过 getter 方法访问属性 high 和 low 也有一点副作用。如果把属性名拼错，会引发一个 NoMethodError 异常。使用哈希时不会有这个问题，因为在哈希中试图访问非法的键只 会返回 nil 而不会引发异常
<br>Struct::new 方法还能接收一个可选的块，在新类的上下文中被评价执行。这很拗口，但它实质上是说，我们能在块中定义实例方法和类方法。
```
class  AnnualWeather
  Reading = Struct.new(:high, :low) do 
    def average
      (high + low) / 2.0
    end
  end
  def initialize
    @readings = [Reading.new(2,1), Reading.new(2,1), Reading.new(2,1), Reading.new(2,1)]
  end
  def average
    return 0.0 if @readings.size.zero?
    total = @readings.reduce(0.0) do |sum, reading|
      sum + reading.average
    end
    total / (@reading.size.to_f)
  end
end
x = AnnualWeather.new
puts x.average 
```
### 总结： 
##### 在处理结构化数据时，如果创建一个新类不那么合适时，推荐使用 Struct 而非 Hash。
##### 将 Struct::new 的返回值赋给常量，并像类一样使用它。

### 三、通过在模块中嵌入代码来创建命名空间
下面的类定义有什么问题呢？
```
class Binding
  def initialize (style, options = {})
    @style = style
    @options = options
  end
end
```
如果Binding类恰好已经被创建过了，这段代码就不是定义一个新类，而是打开了本就存在Binding类并修改了它。
Ruby中通过命名空间的作用域将名字进行分割。
命名空间是一种保证常量唯一性的方式，它最基本的功能就是创建作用域从而定义互不冲突的常量。
所有的核心 Ruby 类都存在于被称为“全局”的命名空间内，当你 不加命名空间地定义类时，这个类就被放在了全局命名空间里。同时存在与已有名字发生冲突的风险。

定义命名空间只需要将类定义嵌入在一个模块中(该模块在之前未定义过)：
```
例如 
module Notebooks
  class Binding
    ....
  end
end
 ```
<br>把类定义嵌入模块中使 Binding 类和核心类区别开来，就避免了同名造成的困扰。
<br>引用新类需要引用模块名并使用“类路径分隔符” 即 “::”, 例如 Notebooks::Binding
<br>使用命名空间时，常用的实践是在项目的文件系统上使用和命名空间一样的目录结构。比如，上述类的定义应该能在 notebooks 文件夹下的 bindings.rb 文件中找到。换句话说，名字 Notebooks::Binding 映射着文件 notebooks/bindings.rb。
<br>假如用来创建命名空间的模块在之前已经定义过，那么可以在类定义中直接使用模块名和类路径分隔符，即：
```
class Notebooks::Binding
  ....
end
```
<br>在入口源文件中定义了命名空间模块， 随后加载所有剩下的源文件。但是如果在没有预先定义命名空间时就尝试使用这种语法会导致一个 NameError 异常。Ruby 如果找不到引用的常量就会引发这个异常。

#### Ruby中如何查找常量？
<br>检查当前词法作用域和所有闭包词法作用域。如果无法找到这个常量，Ruby 将循着继承体系继续寻找。
<br>谈到命名空间，我们最关心的就是词法作用域，表示实际定义或引用常量的位置。
```
module Demo
  KEY = "111111"
  class Test
    def initialize (key = KEY)
      ....
    end
  end
end
```
KEY和Test在同一个词法作用域中，所以 initialize 方法能够在不加限定符（使用常量全路径）的情况下就使 用常量 KEY。
```
module Demo
  KEY = “111111”
end

class Demo::Test
  def	initialize (key = Key) //这里会出现NameError
    ....
  end
end
```
<br>这一次命名空间和词法作用域都使用模块来定义，不过它们都在创建完常量 KEY 之后立即关闭了。
<br>类定义在正确的命名空间里，只是它并没有和 KEY 共享同一个词法 作用域，因此不能不加限定符就使用它。因为 Ruby 在词法作用域或其继承体系中无法 找到常量 KEY，所以这段代码会引发 NameError 异常.
<br>修复的话可以直接加上限定符， 即：
```
class Demo::Test
  def	initialize (key = Demo::Key)
    ....
  end
end

module Cluster
  class Array
    def initialize (n)
      @disk = Array.new(n) {|i| "disk#{i}"}
    end
  end
end
```
<br>上面的代码定义了 Cluster::Array 类，它需要用到顶级类 Array。根据常量的检索 规则，当前上下文中没有限定符的常量 Array 表示类 Cluster::Array，而这并不是我们期望的。解决方法是：为常量 Array 加上限定符。由于这是顶级常量，我们知道它们被存 储在类 Object 中，因此限定符常量的名字将是 Object::Array。这看起来有点怪，因此 Ruby 允许我们简写为 ::Array。
```
module Cluster
  class Array
    def initialize (n)
      @disk = ::Array.new(n) {|i| "disk#{i}"}
    end
  end
end
```

