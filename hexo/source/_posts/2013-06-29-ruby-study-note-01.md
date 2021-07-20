---
layout: post
title: "Ruby学习笔记-初步涉猎"
date: 2013-06-29 09:49:23 +0800
comment: true
categories: 林中笔记
---

闲来无事，上codecademy学下Ruby。
尊贵的红宝石，美好而瑰丽，让人不忍一睹风采……好吧我太废话了。
根据codecademy的介绍，Ruby的特性非常明显:
- 高级语言（High-level）
- 解释型语言（Interpreted）
- 面向对象（Object-oriented）
- 简单易用（Easy to use）

当然……作为一种面向对象的语言，这些特点几乎是必须的……
话说……我不知道ruby算不算脚本语言。

--
##变量
ruby的变量定义和数据类型真是轻便快捷，比较类似javascript吧，弱类型。
变量是无需定义的，抑或称之为动态数据类型吧。
比如

	i = 1
	s = "string"
	arr = [1,2,3]
真是方便得要死。=_=

跟其他面向对象语言类似，ruby的所有数据类型都是对象。

##操作符
操作符基本跟其他语言相似。有几个不同的地方，记录一下：
10 ** 2  #相当于10的2次方 这种写法也挺好的

##字符串（string）
有两种形式： `"string"` 和 `'string'`
比较方便的是，字符串内变量的引用：

    favorite_language = "Ruby"
    puts "My favorite language is #{favorite_language}!"
    My favorite language is Ruby!

另外，字符串带了很多函数，可以方便的对字符串进行各类操作：

    my_name = "eric"
	my_name.upcase #转换大小写
	=> "ERIC"
	my_name.capitalize
	=> "Eric"
	my_name.reverse #反转
	=> "cire"

puts（put string的简写）和print的区别：

	3.times { puts "Hello!" }
	Hello!
	Hello!
	Hello!
	3.times { print "Hello!" }
	Hello!Hello!Hello!

也就是一个是带换行的一个不带。

#逻辑运算

我发现Ruby的很多基本语句都要以`End`结尾的……感觉似乎不太有必要吧。

最基本的IF语句：

	if condition
	  # Do something
	end

除了elsif比较奇怪外……其他的还是很规矩

	if x > y
	  puts "x is greater than y!"
	elsif x < y
	  puts "x is less than y!"
	else
	  puts "x equals y!"
	end

然后是比较奇特的unless语法

	unless x == 10   #其实就等同于 if x!=10 感觉没少几个字- -不解为什么要加入这个语法。
	  puts "I get printed!"
	end

##方法
方法定义跟其他语言也大同小异

	def method_name(arguments)
	  # Code to be executed
	end

比较困惑的是返回值的问题，不过找到了[答案](http://imshanks.com/ruby-methods-return-valu/)

##循环语句
ruby的循环方式还是很多的：
-while

	while true
	  puts "I'm an infinite loop!"
	end

-until

	until counter == 0
	  puts "Counter currently at #{counter}."
	  counter -= 1
	end

-for

	for number in (0..5) #这里，..表示包括5，...表示不包括
	  puts number
	end

还有遍历数组

	for item in my_array
	  puts item
	end

-.times

	3.times { puts "Chunky bacon!" }

这样就实现了输出三个字符串，方便得要死

-.each

	one_to_ten.each do |num|
	  print (num**2).to_s + " "
	end

##其他

-数组的操作：

	y.push("cartoon foxes") #增加一个元素

	z = Array.new(3, "Matz!") #构造方法更灵活地创建数组

-哈希（hash）

	my_hash = {
	  :key1 => "First value!",
	  :key2 => "Second value!",
	  :key3 => "Third value!"
	}
	my_hash[:key2]
	=> "Second value!"

所谓哈希……其实就是字典吧，键值对的数据结构。

