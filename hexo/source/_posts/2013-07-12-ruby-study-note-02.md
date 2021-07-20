---
layout: post
title: "Ruby学习笔记-杂记"
date: 2013-07-12 09:49:23 +0800
comment: true
categories: 林中笔记
---

比较零散的记录

--
##array操作

`.push` 很明显就是添加一个元素

`<<` 添加元素另一种写法

		"Yukihiro " << "Matsumoto"
		# ==> "Yukihiro Matsumoto"

`||=` 这个语法也挺奇怪，意思如下

	prime_array ||= [] 这句的意思等于 
	prime_array = [] if prime_array.nil?

`yield`关键字 算是占位符，或者有说类似虚函数，后面才实现

	def greeter #
	    yield
	end
	phrase = Proc.new {puts "Hello there!"}
	greeter(&phrase)
 
`collect` or `map` or `select` 这三个的作用。。算是迭代器吧c操作数字数组等，map操作字符串数组……不过我暂时不知道他们的区别。


`lambda` 匿名函数，相当于一个block 
`is_a?[类型名]` 这个语法就是判断一个变量是否是某种类型 有种isSubClass的概念 

	my_array = ["raindrops", :kettles, "whiskers", :mittens, :packages]
	symbol_filter = lambda {|x| x.is_a? Symbol}
	symbols = my_array.select(&symbol_filter)

`attr_accessor`  `attr_reader` `attr_writer` 三种属性，或者说访问器，也就是给变量加上一些访问设置，主要用在类里边的~

先到这里，以后慢慢补充~