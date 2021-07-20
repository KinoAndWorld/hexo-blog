---
title: 从零开始开发一个App（2）- 简易REST API服务端
date: 2016.08.03 13:33:20
tags:
categories: 技术杂谈
---

上篇文章讲到使用scrapy爬虫轻而易举地爬取到了我们需要的数据内容，并且已经保存在本地文件中。这一篇，我们换一个姿势，开启rails服务端之旅。

### 安装与初始化

rails环境的安装，称不上特别简单，但也不难。我也是照本宣科的，甩一个教程就好 [如何快速正确的安装 Ruby, Rails 运行环境](https://ruby-china.org/wiki/install_ruby_guide)

安装好之后，跟scrapy爬虫一样，我们只需要在命令行用一行代码创建工程。

首先创建一个`yx_review_server`目录并进入，
然后

```shell
rails new yx_review
```
稍候片刻，一个完整的工程拔地而起，巍峨地竖立在终端之巅。

![Atom IDE](http://upload-images.jianshu.io/upload_images/25038-5d1c003336c618d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
### 数据导入
首先，跟爬虫一样，我们先建一个model，字段照搬。而在rails中这个任务只需要一行代码就可以完成：

```
rails g model review title cover_image summary score
```
更令人亦可赛艇的地方就是，rails中的model与数据库是无缝连接的，只要配置好数据库设置，model的创建与获取等在底层与数据库交互的细节是完全透明的。

创建好model之后，我们打开目录下的`db/seed.rb`文件，将之前的json文件作为数据库初始种子，写入数据库中：

```ruby
local_path = "/Users/Kino/Development/YouxiaReview/yxReview/items.json"
records = JSON.parse(File.read(local_path))
records.each do |record|
  Review.create!(title: record['title'],
    cover_image: record['cover_image'],
    summary: record['summary'],
    score: record['score'])
end

```

为了简单起见，这里的local_path我直接用本地的绝对路径。
接下来，我们在终端输入

```
rake db:migrate

rake db:seed
```
分别进行数据库的迁移与种子数据的载入，相当于根据model在本地创建数据库并写入一些初始数据。

如果需要测试数据库是否成功创建并写入数据，可以在终端输入`rails console`然后输入`Review.all`查看是否打印出对应的数据。

到这一步，前期准备工作就正式完成了。

--- 

### API接口
所谓工欲善其事必先利其器，在rails中，有个叫`grape`的库专门用于API接口的编写REST API，于是我们先装为敬。
打开根目录下的Gemfile文件，并添加一行`gem 'grape'`

然后终端中输入`bundle install`进行安装。

安装完成后，在编写api之前，我们先在`config/application.rb`中加入下面两行，以便rails程序能识别并载入我们的api文件夹

```ruby
config.paths.add File.join('app', 'api'), glob: File.join('**', '*.rb')
config.autoload_paths += Dir[Rails.root.join('app', 'api', '*')]
```

然后进入app目录，创建api目录并创建`api.rb`如下：

![api目录结构](http://upload-images.jianshu.io/upload_images/25038-566852cd607b74e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`api.rb`编辑内容如下

```ruby
class API < Grape::API
  format :json
  prefix 'api'
  version 'v1', using: :path

  resource :reviews do
    desc "List all review"
    get do
      Review.all
    end
  end
end
```

这里声明了数据格式为json，并且指定resource为reviews，使用get方法返回数据库中所有的`Review.all`对象。

由于目前的数据量非常少，我这里为了简（tou）单（lan）直接返回了所有数据，在一般情况下都是需要排序和分页的，大致是这个样子：

```ruby
params do
  requires :page, :type =>Integer, :desc => "page"
  requires :size, :type =>Integer, :desc => "size"
end

get do
  Review.order(created_at: :desc).page(params[:page]).per(params[:size])
end
```


在这个时候rails的路由还没能识别到api接口，我们需要在根目录下的`config/routes.rb`中加入

```ruby
mount API => '/'
```
然后一切准备就绪，我们在终端中输入 `rails server`将服务器跑起来。

如无意外，rails将在本地建立一个监听端口3000的服务端，我们在浏览器中输入
`http://localhost:3000/api/v1/reviews`

![浏览器访问截图](http://upload-images.jianshu.io/upload_images/25038-76f1936ec87800e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 里程碑

作为一个极简实现，它的本身存在诸多不完善，比如api分版本分结构的需要，比如接口带参数以及验证的需要等等。但是对于一个iOS开发者以及服务端小白来说，这是具有里程碑的意义的，下一篇，回到iOS的主战场，快速构建一个炫酷(并不)的App。

--- 

参考Link：

[如何快速正确的安装 Ruby, Rails 运行环境](https://ruby-china.org/wiki/install_ruby_guide)

[Ruby JSON](http://www.runoob.com/ruby/ruby-json.html)

[Ruby on Rails 實戰聖經](https://ihower.tw/rails4/index.html)

[Build Great APIS with Grape](https://www.sitepoint.com/build-great-apis-grape/)


---

**系列链接**

{% post_link 从零开始开发一个App（1）-Scrapy爬虫 %}
{% post_link 从零开始开发一个App（2）-简易REST-API服务端 %}
{% post_link 从零开始开发一个App（3）-iOS客户端 %}