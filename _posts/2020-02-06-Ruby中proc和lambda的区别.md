## Ruby中的block
在Ruby代码中会用到大量的block(代码块), 你可以通过 {...} 或者 do...end 来定义 block, 例如:
```ruby
10.times do
	puts 'a'
end
# 或者
(1..10).each {|i| puts i * 2 }
```
Ruby中的block, 就是匿名函数, 而这个匿名函数是Ruby世界中的一等公民. 实际上Ruby中每个方法在执行时, 都会检测有没有传递block. 而方法可以通过 yield去调用匿名函数.例如:
```ruby
def a_method(a, b)
	a + yield(a, b)
end

a_method(1, 2) {|x, y| (x + y * 2) * 3 }
# => 16
```
当没传递block时, yield会报错, 你可以在方法加入 Kernel#block_given? 来判断有没传递block.
```ruby
def a_method(a, b)
	if block_given?
		a + yield(a, b)
	else
		'no block given'
	end
end
```
block不是Ruby的对象, Ruby中提供了两种方式把block转换成Proc对象: proc 和 lambda, Proc对象可通过Proc#lambda? 判断是否 lambda.
```ruby
p = Proc.new {|x|  x + 1 }
l = lambda {|x| x + 2}
# lambda {|x| x + 2} 也可以写成 ->(x) { x + 2 }, 两者完全一致
p.lambda? # => false
l.lambda? # => true
```
##proc 和 lambda 的区别
(1) block中return关键字的作用域
lambda中return作用域只限于block内部, 也就是block外部往下的代码还会往下执行;
proc中return作用域则是定义这个proc的作用域, block外部往下的代码就不会执行了;
```ruby
def method_l
	lambda { return 10 }.call
	return 20
end

def method_p
	Proc.new{ return 10 }.call
	return 20
end

method_l # => 20
method_p # => 10	
```
(2) 检测参数的方式
新建一个proc对象, 带2个参数, 当向proc传递3个或1个参数时, 多余参数会被忽略, 不足的参数则默认为nil, 然后继续执行.
```ruby
p = Proc.new{|x, y| [x, y]}
p.arity #=> 2

p.call(1, 2, 3) # => [1, 2]
p.call(1) # => [1, nil]
```
那么lambda对象呢, 结果会报 ArgumentError 错误.
```ruby
l = lambda{|x, y| [x, y]}
l.arity # => 2

l.call(1, 2, 3) # => ArgumentError
l.call(1) # => ArgumentError
```
也就是说, lamdba对参数严格, 而proc则较为宽松. 

那日常代码中proc和lambda该选择哪个用呢? 我们先看看日常用的很多的迭代器中的block是哪种类型吧.
```ruby
(1..3).each do |x, y|
	puts "x: #{x}, y: #{y}"
end
# => x: 1, y:
# => x: 2, y:
# => x: 3, y:
```
并未报错, 可见是proc. 迭代器应用范围广, Array/Hash等都能用, 对参数宽容度要求高, 因此用proc.
不过因为proc中的return会影响到定义proc的作用域, 因此要注意, 迭代器block中不要写return, 这种发生bug时比较难查.
```ruby
def a_method
	2.times{|i| return i}
	return 10
end
a_method # => 0
```
一般来说, lambda对参数严格, 比较接近一般的方法, 并且方便通过return退出. 除非刻意要使用proc的特性, 日常代码中都建议使用 lambda. 