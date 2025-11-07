---
title: Pipenv 使用指南
date: 2018-01-04T02:47:34+08:00
categories: [Python]
tags: [Python]
---
pipenv是requests作者的一个项目, 整合了virtualenv, pip的优点，用于更方便地为项目建立虚拟环境并管理虚拟环境中的第三方模块。后来这个项目交给Python基金会维护。
<!--more-->
为什么要使用Pipenv？：
* 自动关联项目相关的 virtualenv，能够快速的加载 virtualenv。
* 提供的 pipenv替代 pip并自带一个依赖清单 Pipfile，和依赖锁定 Pipfile.lock。
* 其中 Pipfile除了依赖清单还支持固定 pypi源地址,固定 python版本。
* Pipfile还支持 dev依赖清单. pipenv install的包会强制使用 Pipfile中的源.
* 解决了 pip install pandas时里的 numpy依旧走官方 pypi.
还有就是可以直接切换 python2,3
* 使用 pipenv graph命令可以看到依赖树

#### 1、安装Pipenv
    pip3 install pipenv

#### 2、创建虚拟环境
pipenv管理虚拟环境是按项目来的, 要为你的某个项目新建一个虚拟环境, 只需要在项目目录下运行如下命令:
```sh
    # 新建Python2 的虚拟环境
    pipenv --two
    
    # 新建Python3 的虚拟环境
    pipenv --three
    
    # 更换豆瓣的pipy源
    sed -i s/pypi.python.org/pypi.doubanio.com/g Pipfile
```

该命令会在项目目录下生成一个`Pipfile`的文件，用于记录虚拟环境的信息及第三方依赖的信息。

#### 3、安装第三方模块
pipenv可以自动安装你项目的第三方模块 :
    
    pipenv install

安装列表是通过读取`pipfile`, `pipfile.lock`文件实现的, 如果没有这两个文件就根据`requirements.txt`生成`pipfile`和`pipfile.lock`并读取.
如果想单独安装某个模块还可以指定模块名安装:
    
    pipenv install sanic

单独安装模块后会自动将新模块信息添加到pipfile中, 要同时更新pipfile.lock需要运行:

    pipenv lock

#### 4、进入虚拟环境

    # 进入虚拟环境
    pipenv shell
    
    # 退出
    exit

还有一个 `pipenv run`的可以直接执行 `virtualenv`环境下的命令。

#### 5、基本命令
```sh
Usage: pipenv [OPTIONS] COMMAND [ARGS]...
Options:
  --update         升级 pipenv, pip 到最新.
  --where          输出项目的目录信息.
  --venv           输出 virtualenv 的目录信息.
  --py             输出 Python 解析器的路径.
  --envs           输出环境变量的设置.
  --rm             删除当前 virtualenv.
  --bare           Minimal output.
  --completion     Output completion (to be evald).
  --man            显示使用手册.
  --three / --two  使用 Python 3/2 来创建 virtualenv
  --python TEXT    直接指定 Python 解析器.
  --site-packages  拷贝系统 site-packages 到 virtualenv.
  --jumbotron      An easter egg, effectively.
  --version        显示版本信息并退出.
  -h, --help       显示当前信息并退出.
Commands:
  check      检查安全漏洞和反对 PEP 508 标记在Pipfile提供.
  graph      显示当前依赖关系图信息.
  install    安装提供的包，并加入 Pipfile 的依赖清单中
  lock       生成 Pipfile.lock.
  open       在编辑器(vim)查看一个特定模块.
  run        在 virtualenv 中执行命令.
  shell      切换到 virtualenv 中.
  uninstall  删除提供的包，并清理 Pipfile 的依赖清单中.
  update     卸载当前所以依赖，然后安装最新包
```

