Title: Pelican 初始化和CI自动化部署 
Date: 2019-08-13
Category: pelican

Pelican 是一款基于Python语言的静态网站生成程序, Pelican本身的使用非常简单。托管到github或者gitlab等代码托管网站可以自动生成响应的页面，非常方便。但是如果要生成对应的页面，在写完内容之后，需要生成静态网站内容，我们还需要把源码和生成的静态页面分开上传到仓库的不同分支，(github 是需要单独新建分支，gitlab则不需要)。如果我们可以让机器自动完成最后一步，我们只管内容写完，上传到仓库该多好。CI工具就是来做这一步的。本文主要的内容分为以下部分

* pelican的基本命令和初始化
* 基于githab 和 的CI配置
* 基于gitlab 的CI配置

## Command Cheatsheet
````
pelican content
pelican --listen
````

## 初始化

初始化方案请严格遵守[Quickstart — Pelican 3.6.3 documentation](http://docs.getpelican.com/en/3.6.3/quickstart.html)文档的内容。相关的命令如下
```
pip install pelican markdown  # 如果已经安装请忽略，python2、python3均可
mkdir -p ~/projects/yoursite
cd ~/projects/yoursite
pelican-quickstart
```
quickstart 中会需要回答下面的这些问题
```
> Where do you want to create your new web site? [.]
> What will be the title of this web site? # your web site name
> Who will be the author of this web site? # author name
> What will be the default language of this web site? [en] 
> Do you want to specify a URL prefix? e.g., https://example.com   (Y/n) y
> What is your URL prefix? (see above example; no trailing slash) # https://example.com
> Do you want to enable article pagination? (Y/n) y
> How many articles per page do you want? [10] 10
> What is your time zone? [Europe/Paris] Asia/Shanghai
> Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n) y
> Do you want to upload your website using FTP? (y/N) n
> Do you want to upload your website using SSH? (y/N) n
> Do you want to upload your website using Dropbox? (y/N) n
> Do you want to upload your website using S3? (y/N) n
> Do you want to upload your website using Rackspace Cloud Files? (y/N) n
> Do you want to upload your website using GitHub Pages? (y/N) y
> Is this your personal page (username.github.io)? (y/N) n
Done. Your new project is available at /dir/path
```
注意事项：

* 关于时区，如果不用上面提到的，请根据下面的[wiki链接](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)选择对应的时区


新建文档
在 content 目录下新建markdown文件,注意头部的写法
```
Title: My First Review
Date: 2010-12-03 10:20
Category: Review

Following is a review of my favorite mechanical keyboard.
```
执行`pelican content`注意要在项目的根目录下执行,会自动生成对应的静态文件。注意pelican的默认输出文件夹是output.(因为后面的gitlab的文件夹有不同，这里先提示，下面会再次提到)

测试效果
注意，这里的命令和文档中的有区别，文档中是过去的做法，现在推荐的是下面的命令`pelican
--listen`,打开网页输入 http://localhost:8000/ 可以看到本地的效果。

以上是最基本的pelican的应用。

## 设置仓库
一般我们会把内容托管在代码仓库平台上，常用的选择是github或者gitlab，国内可以选择gitee。需要注意，为了和后面的CI部分结合，github和gitlab的处理方式略有不同，会在下文说明。如果没有特殊说明，则两处的处理方案是一致的。

仓库的初始化大同小异，不在这里赘述。

### set .gitignore file
gitignore 文件是有必要设置的，因为一般在output中会保留生成后的静态文件，从最终自动化部署的角度来看，这些文件是没有必要的。另外还有一些python的中间文件，vim异常操作导致的swp后缀文件或者swo后缀文件等。一般gitignore 文件可以参考[这里](https://github.com/getpelican/pelican-blog/blob/master/.gitignore)。这里给出自己的实践结果

````
*~
._*
*.lock
*.DS_Store
*.swp
*.out
*.py[cod]
output
````

仓库设定好之后，建议增加README.md文件，对内容做简要说明。上传仓库的操作不再赘述。一般选择master分支上传。

## 自定义域名
假设我们有一个自定义的域名叫xx.yy.zz,在github中通过CNAME文件增加域名和域名解析做相应的设置。gitlab中则是在仓库的设置里做配置和域名解析做设置实现，两者略有区别。

### gitlab 域名认证
参考[Index · Custom domains ssl tls certification · Pages · Project · User ·
Help · GitLab](https://gitlab.com/help/user/project/pages/custom_domains_ssl_tls_certification/index.md#4-verify-the-domains-ownership)， 这里提到的DNS record 和 Verification status的内容的填写会根据你的域名解析供应商略有区别。

````
xx.yy.zz CNAME namespace.gitlab.io.
_gitlab-pages-verification-code.xx TXT code
````
gitlab为了验证域名的归属真实性，要要求填写上面提到的txt部分的内容。其中code部分的内容是gitlab提供的语句最后的验证码部分，注意不要把整个语句都填写进去。

## CI/CD

关于CI、CD的概念不是本文的重点，不做详述，只需要知道基于这个概念，可以实现我们的代码上传到仓库后自动的将内容部署并且可以在网页端展示即可。有兴趣的小伙伴可以从下面的链接找到线索，自行探索。
 * [什么是 CI/CD？](https://www.redhat.com/zh/topics/devops/what-is-ci-cd)
 * [Continuous integration - Wikipedia](https://en.wikipedia.org/wiki/Continuous_integration)

Github 默认没有自带CI的功能（据说很快会有该功能），需要用Travis CI协助实现。Gitlab自带CI功能，但是两者的配置方式不同，会在后面细说。

### Github & Travis CI

自动化部署有很多工具，基于github 比较流行的是Travis CI.另外，基于Travis CI部署
github pages时也会有多种方案。本文参考的是这篇[文章](https://blog.m157q.tw/posts/2016/05/08/use-travis-ci-to-publish-pelican-blog-on-github-pages-automatically)。
也可以考虑不使用ghp-import，把部署需要的相关命令直接写在.travis.yml中。
总的来说，要想实现自动化部署，需要了解yml文件、Makefile相关知识，还需要一些虚
拟环境（比如docker），shell、git等一系列的知识点

#### 设定CI
公开仓库的CI在[Travis CI - Test and Deploy Your Code with Confidence](https://travis-ci.org/)，私有仓库在[Travis CI](https://travis-ci.com/)

#### 初始化准备

设置Token	    参考上面的文章设置

setup orphan branch 注意：在设置分支，且删除所有内容时，不要误删git文件，特别
		    是gitignore文件。

设置git的名称	    git config user.name "Mona Lisa"
		    https://help.github.com/en/articles/setting-your-username-in-git
		    去掉--global 参数

必要的包 	    ghp-import https://github.com/davisp/ghp-import

#### 设定.travis.yml
````
language: python  
python:  
- '3.6' 

branches:  
  only:  
  - master
         
install:  
- pip install -r requirements.txt  # 這邊其實可以直接寫死 pip install ${package}  
                                   # 使用 requirements.txt 純粹是我個人喜好  
script:  
- make travis  # 需要在 Makefile 新增 travis 的 label  
````
#### 设定 requirements.txt
````
-i https://mirrors.aliyun.com/pypi/simple
pelican==4.1.0
ghp-import==0.5.5
Markdown==3.1.1
````
#### Makefile
````
PY?=python3
PELICAN?=pelican
PELICANOPTS=

BASEDIR=$(CURDIR)
INPUTDIR=$(BASEDIR)/content
OUTPUTDIR=$(BASEDIR)/output
CONFFILE=$(BASEDIR)/pelicanconf.py
PUBLISHCONF=$(BASEDIR)/publishconf.py

GITHUB_PAGES_BRANCH=gh-pages

GITHUB_REPO_SLUG=repo_name/xx.yy.zz
GITHUB_REMOTE_NAME=origin 
# 以上參數請根據需求自行替換  
GITHUB_COMMIT_MSG=$(shell git --no-pager log --format=%s -n 1)  


DEBUG ?= 0
ifeq ($(DEBUG), 1)
	PELICANOPTS += -D
endif

RELATIVE ?= 0
ifeq ($(RELATIVE), 1)
	PELICANOPTS += --relative-urls
endif

travis: publish  
    # 為 Travis CI 設定 git 的 user.name 和 user.email  
    # 沒設定 email 的話，GitHub 上面看到的 Author 會是 Unknown  
	git config user.name "proj_name"  
	git config user.email "proj_email"
	echo 'xx.yy.zz' > ./output/CNAME


    # 將 Pelican output dir 的內容 commit 到 GitHub Pages 用的 branch，準備 push 上去  
    # 因為我用的是 user site，所以 branch 是 master。如果是 project site 的話，branch 會是 gh-pages  
	ghp-import -n -r $(GITHUB_REMOTE_NAME) -b $(GITHUB_PAGES_BRANCH) -m "$(GITHUB_COMMIT_MSG)" $(OUTPUTDIR)  

    # 將剛剛的 commit force push 到 GitHub 上相同的 branch  
    # 不用 -f (force push) 的話一定會因為 conflict 而失敗  
    # 因為每次 Travis CI build 只會有一個 commit  
    # 而且該 branch 只會存一堆靜態檔案，每次變動都很大，沒有啥需要保存 commit log 的必要性。  
	@git push -fq https://${GH_TOKEN}@github.com/$(GITHUB_REPO_SLUG).git $(GITHUB_PAGES_BRANCH):$(GITHUB_PAGES_BRANCH) > /dev/null  
    # 用 @ 可以讓 Travis CI 不要顯示這行在 log 上，這樣別人就不會看到你的 GitHub Personal Access Token 了，也就是這裡用的 ${GH_TOKEN}  

help:
	@echo 'Makefile for a pelican Web site                                           '
	@echo '                                                                          '
	@echo 'Usage:                                                                    '
	@echo '   make html                           (re)generate the web site          '
	@echo '   make clean                          remove the generated files         '
	@echo '   make regenerate                     regenerate files upon modification '
	@echo '   make publish                        generate using production settings '
	@echo '   make serve [PORT=8000]              serve site at http://localhost:8000'
	@echo '   make serve-global [SERVER=0.0.0.0]  serve (as root) to $(SERVER):80    '
	@echo '   make devserver [PORT=8000]          serve and regenerate together      '
	@echo '   make ssh_upload                     upload the web site via SSH        '
	@echo '   make rsync_upload                   upload the web site via rsync+ssh  '
	@echo '   make github                         upload the web site via gh-pages   '
	@echo '                                                                          '
	@echo 'Set the DEBUG variable to 1 to enable debugging, e.g. make DEBUG=1 html   '
	@echo 'Set the RELATIVE variable to 1 to enable relative urls                    '
	@echo '                                                                          '

html:
	$(PELICAN) $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS)

clean:
	[ ! -d $(OUTPUTDIR) ] || rm -rf $(OUTPUTDIR)

regenerate:
	$(PELICAN) -r $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS)

serve:
ifdef PORT
	$(PELICAN) -l $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS) -p $(PORT)
else
	$(PELICAN) -l $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS)
endif

serve-global:
ifdef SERVER
	$(PELICAN) -l $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS) -p $(PORT) -b $(SERVER)
else
	$(PELICAN) -l $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS) -p $(PORT) -b 0.0.0.0
endif


devserver:
ifdef PORT
	$(PELICAN) -lr $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS) -p $(PORT)
else
	$(PELICAN) -lr $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS)
endif

publish:
	$(PELICAN) $(INPUTDIR) -o $(OUTPUTDIR) -s $(PUBLISHCONF) $(PELICANOPTS)

github: publish
	ghp-import -m "Generate Pelican site" -b $(GITHUB_PAGES_BRANCH) $(OUTPUTDIR)
	git push origin $(GITHUB_PAGES_BRANCH)


.PHONY: html help clean regenerate serve serve-global devserver publish github
````



### Debug

如何debug，进入https://travis-ci.com/查看部署的记录，结合页面展示出来的结果查
问题。 文档：https://docs.travis-ci.com/

在配置完成后的前五次部署全部未通过，后面的5次解决了一些关键问题。这里倒叙记录
如下：

````
pelican /home/travis/build/repo_name/xx.yy.zz/content -o /home/travis/build/repo_name/xx.yy.zz/output -s /home/travis/build/repo_name/xx.yy.zz/publishconf.py
WARNING: Feeds generated without SITEURL set properly may not be valid
WARNING: No valid files found in content for the active readers:
  | BaseReader (static)
  | HTMLReader (htm, html)
  | RstReader (rst)
Done: Processed 0 articles, 0 drafts, 0 pages, 0 hidden pages and 0 draft pages in 0.07 seconds.
````
在本地content中有一篇文章，但是这里始终不展示。最终发现是没有安装markdown导致
的。本地`pip install markdown` 要写到requirement.txt文件中去。利用
`pip show markdown` 获取对应的版本号
参考文档：https://github.com/getpelican/pelican/issues/1868

````
$ make travis
Makefile:32: *** missing separator.  Stop.
The command "make travis" exited with 2.
````
Makefile在执行的命令前用tab，不能直接用四个空格。把对应的命令前的格式调整即可。
参考文档：https://stackoverflow.com/questions/16931770/makefile4-missing-separator-stop

现象：打开对应的域名出现404的情况
原因：检查Github的gh-pages分支发现CNAME没有设置。需要在Travis CI的某个步骤增加
      对应的语句`echo 'xx.yy.zz' > ./output/CNAME`该语句需要增加在上传到
      远程仓库之前
参考：http://www.wisdomofjim.com/blog/creating-a-cname-domain-file-on-a-travis-ci-server

````
3.7 is not installed; attempting download
Downloading archive: https://storage.googleapis.com/travis-ci-language-archives/python/binaries/ubuntu/14.04/x86_64/python-3.7.tar.bz2
$ curl -sSf -o python-3.7.tar.bz2 ${archive_url}
curl: (22) The requested URL returned error: 404 Not Found
````

Python 版本的问题，导致在虚拟环境准备时找不到python3.7。建议不要用最新的python
，不稳定的同时，也会偶尔出现不兼容的问题。还是退回一个版本比较安全。将.travis.yml 中的
python 版本降到3.6解决了这个问题。
参考文档： https://github.com/travis-ci/travis-ci/issues/9069
	   https://github.com/travis-ci/travis-ci/issues/9815




### Gitlab & CI

gitlab的CI是集成在gitlab仓库中的，如果用习惯了github，初次切换过来会稍有不适应。而且两者的配置差异还是比较大的。总的来说有如下的几个注意点
* gitlab pages 不像github，不需要单独新建类似于gh-pages的分支存放静态页面的内容。
	* gitlab 只需要在master分支下有public文件夹存放对应的内容即可。而public文件夹的内容又不需要在仓库中建立，因为CI可以自动建立和部署。所以在CI文件配置时需要注意。
	* 参考：[GitLab Pages | GitLab](https://docs.gitlab.com/ee/user/project/pages/)
* gitlab的CI配置原理和github相同，但是具体的配置方法有差异，建议直接参考gitlab提供的示例来做。
	* [GitLab Pages examples · GitLab](https://gitlab.com/pages?page=2) 这里有各种主流的静态页面生成工具的案例。我们只需要找到pelican的文件即可
	* 在Pelican的案例的readme文件中已经说明需要修改的文件内容，根据要求调整即可。

### debug
理论上到这一步就已经完成配置。为了查看结果，我们可以上传文章尝试一下。不过初期尝试，不要指望一次通过，有问题是常态，关键在如何解决问题。

在gitlab 的仓库列表页面就可以看到结果，如果是所在行有绿色的钩就是通过，如果是红色的叉，说明出现了问题。进入项目后，左侧选择CI/CD，Jobs 向下可以看到每一次的执行情况，选择最近的一次执行，点开就可以看到虚拟环境里执行的情况了。以下面的报错为例（这也是我实际遇到的真实案例）：

````
pelican /builds/xx.yy.zz/content -o /builds/xx.yy.zz/output -s /builds/xx.yy.zz/publishconf.py
WARNING: Docutils has no localization for 'cn'. Using 'en' instead.
Done: Processed 1 article, 0 drafts, 0 pages, 0 hidden pages and 0 draft pages in 0.32 seconds.
Uploading artifacts...
WARNING: public/: no matching files

ERROR: No files to upload
````
这里显示pelican的content命令已经生成静态页面，但是找不到public的文件夹。前文提到过，pelican默认的输出文件夹叫output，而gitlab需要在public文件夹下存放静态页面才能生成对应的gitlab pages.所以需要回到前面的配置文件去调整一下。（如果这里按照样例配置都是正确的，我是自己做了尝试才导致的错误）

## 其他

关于Pelican的应用，上面只是最基本的内容，还可能涉及到如下的一些话题，本文仅做基本的解答或者提供一些思路。如果以后有机会再撰文详细描述。

 * 关于主题
	* Pelican提供了一些主题可供选择，相关的配置方法和主题的汇总页面参考下面的链接
		* http://docs.getpelican.com/en/3.6.3/themes.html
		* Themes Collections
			* https://github.com/getpelican/pelican-themes 
			* http://www.pelicanthemes.com/
 * 关于插件
	* https://github.com/getpelican/pelican-plugins
 * 其他配置
	* 增加固定页面
		* http://docs.getpelican.com/en/4.0.1/content.html
	* 修改blogroll或者social link
		* 直接在配置文件中修改即可

Reference
* http://docs.getpelican.com/en/latest/index.html
* [Pelican dev team](https://github.com/getpelican)
* https://blog.m157q.tw/posts/2016/05/08/use-travis-ci-to-publish-pelican-blog-on-github-pages-automatically/
* 其他的可能的方案和资料
  * https://iranzo.github.io/blog/2018/12/07/elegant-website-ci/
  * https://docs.getpelican.com/en/3.0/tips.html
  * http://docs.getpelican.com/en/stable/tips.html
  * http://blog.mathieu-leplatre.info/category/sys.html

## ChangeLog

* 2019-07-15 init
* 2019-08-12 更新和丰富内容
* 2019-08-13 更新和丰富内容
* blog、github、quip、du、ow 发布
* LeonTalk 备份（后续版本以wiki的更新为准）
