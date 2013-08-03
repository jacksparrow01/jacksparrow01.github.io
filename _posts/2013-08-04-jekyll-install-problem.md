---
layout: default
title: jekyll本地安装环境搭建过程中的问题
description: jekyll本地安装环境搭建过程中的问题
---
## 一般的安装流程是这样的：
1. 安装Ruby<br/>
	Jekyll使用动态脚本语言 Ruby 写成。因此在安装Jekyll前，必须先安装Ruby。可以选择直接安装[RubyInstall](http://rubyinstaller.org)，如果是Window系统。

2. 安装DevKit<br/>
	在后面安装一些gem native extension时需要DevKit，下载[DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe](http://rubyforge.org/frs/download.php/76805/DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe)，然后依次执行下面的命令：
	cd [DevKit路径]
	ruby dk.rb init
	ruby dk.rb install

3. 安装jekyll1.1.2版本<br/>
	直接使用ruby的工具gem输入gem install jekyll就可以安装。这里因为网络原因，所以gem安装失败，我是通过下载jekyll.gem以及其所有的依赖的gem一个个进行安装的，可以通过地址[RubyGems](http://rubygems.org/)下载。

4. 运行jerkll<br/>
	安装完后，进入到jekyll的html页面的目录，执行jekyll serve --watch会在本地4000端口打开http请求，使用http://localhost:4000/就可以访问。在这里还碰到了编码的问题：
	<pre><code>E:\xx\jacksparrow01.github.io>jekyll serve
	Configuration file: E:/xx/jacksparrow01.github.io/_config.yml
	       Deprecation: Auto-regeneration can no longer be set from your configurati
	on file(s). Use the --watch/-w command-line option instead.
	            Source: E:/xx/jacksparrow01.github.io
	       Destination: E:/xx/jacksparrow01.github.io/_site
	      Generating... Error reading file E:/xx/jacksparrow01.github.io/_layouts/def
	ault.html: invalid byte sequence in GBK
	Error reading file E:/xx/jacksparrow01.github.io/index.html: invalid byte sequenc
	e in GBK
	  Liquid Exception: invalid byte sequence in GBK in default.html
	error: invalid byte sequence in GBK. Use --trace to view backtrace</code></pre>
	通过修改jekyll的源码可以解决这个问题，具体的步骤如下：<br/>
	a、将文件convertible.rb(去Jekyll安装目录里找)下面的片段（大概31行）进行修改。
	<pre><code>self.content = File.read(File.join(base, name))</code></pre>
	改为：
	<pre><code>self.content = File.read(File.join(base, name),:encoding=>"utf-8")</code></pre>
	b、设置两个用户变量LC_ALL和LANG为en_US.UTF-8。<br/><br/>

5. 参考：<br/>
http://xdutaotao.github.io/blog/2012/05/30/jekyll-encode/<br/>
http://demi-panda.com/2012/12/29/building-sites-with-jekyll/<br/>
http://wowubuntu.com/markdown/<br/>
