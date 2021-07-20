---
layout: post
title: "iOS测试流程简化：通过脚本导出ipa包与上传到Fir"
date: 2015-06-16 14:01:47 +0800
comments: true
categories: iOS
---

###前言

在我们当前的开发流程中，大致分为 **开发-测试-修复Bugs-测试反馈** 这样的循环中，特别是在功能需求频繁修改和UI细节不断调整的节奏中，我们通常需要不停地在Xcode上build-选择profile-archive，然后等待几分钟，导出ipa的包，然后再把包上传到一个版本仓库中，这个仓库或许是自己搭建的服务器，或许是TestFlight、Fir等更专业的内测版本平台，总之通常都需要几个步骤才能全部完成。倘若一日之内需要上传3到4个包，对工作效率的影响尚且不谈，重复同样的事情本身心情就不舒畅。

###祭出利器

要如何做到把上述的N个步骤简化成一步呢，其实很简单。拿Fir为例，Fir提供了可供调用的app上传接口（其他平台我暂时不知道，如果是自己搭建的服务器肯定可以提供上传接口），而且还有好心人把接口写成了脚本（参考资源1），非常nice。但是这样的话还是要自己导出ipa，还是有些繁琐，于是我又找了些许资料，熟悉了`xcodebuild`的用法，还顺带找到了别人写好的另一个脚本（参考资源2），剩下的事情就非常简单了，我合并了二者的脚本，做了一些代码修改，详细的代码可以戳[这里](https://gist.github.com/ec0d8072d3389a9f3b02.git)

这里边有些参数要修改成自己的项目，譬如

- workspace name 
- provisioningProfile 
- scheme name
- 还有Fir需要的AppID和UserToken


配置好之后，把这个脚本放在xcworkspace同级目录，然后终端运行 `./archive_to_upload_fir.sh`
这时候你可以去喝杯咖啡奶茶美年达或者来一套广播体操，在一片宁静中完成了所有繁琐。


###缺点与改进

当然我相信这并不是最终的解决方案，因为脚本还是存在着输出信息不够友好，如果出现编译错误还是要自己另外调试等不便，而且脚本本身还存在可以优化的地方，例如一些变量的参数化，让它变得更通用一些。

---

参考资源：

- [https://gist.github.com/ggshily/8594b69a266a410d82a0](https://gist.github.com/ggshily/8594b69a266a410d82a0)

- [http://blog.csdn.net/vieri_ch/article/details/45147027](http://blog.csdn.net/vieri_ch/article/details/45147027)
