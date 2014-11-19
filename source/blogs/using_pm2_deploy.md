Title: 使用PM2 Deploy部署基于Git版本管理的网站应用
Date: 2014-11-19 13:29
Tags: Node.js PM2 部署

按照官方介绍，PM2是一款用于生产环境Node.js应用进程管理的工具。按照民间介绍，它主要有这样几个功能：保证Node.js应用永远在线（挂掉自动重启）、自动负载均衡、零中断重启应用等。

鉴于它是如此优秀，这里还是简要介绍一下前两个功能。

## 安装

首先，它是一个Node.js写的工具，使用npm即可安装使用：

```
npm install -g pm2
```

## 运行Node.js程序

如果不使用pm2，运行Node.js程序是这样：

```
node xxx.js
```

使用pm2，是这样：

```
pm2 start xxx.js
```

### 监视模式

如果你正在开发Node.js应用，需要在代码变更后自动重启应用，只需要在pm2的参数中加上`--watch`即可：

```
pm2 start xxx.js --watch
```

<!-- $$solo_more$$ -->

### cluster模式

默认情况下pm2是以fork模式启动应用的，如果以cluster模式启动的话，则可以使用pm2自带的负载均衡、零间断重启等功能。

```
pm2 start xxx.js -i 4
```

上面的命令会以cluster模式启动4个应用进程，并自动为它们提供负载均衡，并且可以使用gracefulReload达到更新应用时不中断服务的效果。

> 关于cluster模式，可参见朴灵《深入浅出Node.js》一书。
> 
> pm2低版本默认是以cluster模式启动的。

## 部署应用

这才是本文的重点。PM2的部署功能可以实现网站应用的半自动部署功能。注意本文标题没有加“Node.js”，意味着这个功能并不只适用于Node.js网站应用，事实上它部署功能是用shell写的，跟网站使用什么语言没什么关系。

PM2的部署功能与版本管理工具（Git，不确定是否支持SVN，下文以Git为例）结合比较紧，因此需要保证网站项目使用版本管理工具管理代码，并且服务器可以访问到版本管理服务器。

部署功能是在新版本（0.12？）中才添加进来的。如果你使用的是旧版本的，需要先升级：

```
npm install -g pm2@latest
pm2 updatePM2
```

接下来需要建立一个部署的配置文件，这个文件在本机（操作发布的机器）和服务器上都需要有，因此最好放入Git版本管理中，并且推送到远程代码库（Git服务器）。

切换到项目目录下，然后执行

```
pm2 ecosystem
```

即可得到一个示例json文件（例如我得到的是`ecosystem.json5`），将它做对应的修改，大致如下：

```
{
	"apps" : [{
		"name" : "xxx", //项目的名字
		"script" : "xxx.js",  //项目主入口（Node.js）
		"env": {
			"COMMON_VARIABLE": "true"
		},
		"env_production" : {
			"NODE_ENV": "production"
		}
	}],
	"deploy" : {
		"production" : {
			"user" : "toobug",
			"host" : "server.toobug.net",
			"ref"  : "origin/master", //需要部署的分支
			"repo" : "git@github.com:TooBug/xxx.git",
			"path" : "/var/www/xxx", //web目录
			"post-deploy" : "npm install && pm2 startOrRestart ecosystem.json --env production"
		}
	}
}
```

需要注意：

1. `apps.name`和`apps.script`应该与PM2识别应用有关，后续执行`pm2 restart`的时候可以对应到进程（未证实）
2. `deploy`中可以含有多个环境，需要能够通过SSH（公钥认证）登录服务器
3. web目录并不是真正的放版本库文件的目录，PM2会再建立一个`source`子目录，这个才是真正放代码的目录
4. `post-deploy`是指代码部署完之后执行的命令，这里以Node.js为例子，执行依赖安装，然后重启PM2中的进程

然后就可以使用

```
pm2 deploy ecosystem.json production
```

自动发布网站项目了，非常方便。

```
$>pm2 deploy dev
--> Deploying to production environment
--> on host server.toobug.net
  ○ deploying
  ○ hook pre-deploy
  ○ fetching updates
Fetching origin
  ○ resetting HEAD to origin/master
HEAD is now at eda2cdd xxx
  ○ executing post-deploy npm install && pm2 startOrRestart ecosystem.json --env production
manpath: can't set the locale; make sure $LC_* and $LANG are correct
Now using node v0.11.13
[PM2] restartProcessId process id 0
┌──────────┬────┬──────┬──────┬────────┬───────────┬────────┬─────────────┬──────────┐
│ App name │ id │ mode │ PID  │ status │ restarted │ uptime │      memory │ watching │
├──────────┼────┼──────┼──────┼────────┼───────────┼────────┼─────────────┼──────────┤
│ xxx      │ 0  │ fork │ 7384 │ online │        33 │ 0s     │ 12.438 MB   │ disabled │
└──────────┴────┴──────┴──────┴────────┴───────────┴────────┴─────────────┴──────────┘
 Use pm2 info <id|name> to get more details about an app
  ○ hook test
  ○ successfully deployed origin/master
--> Success

```

在使用过程中还有几个值得注意的点：

- 在部署过程中，PM2会执行一次`git reset --hard`，意味着如果你修改了配置文件之类的，会被还原，因此最好使用环境变量或者新建文件（不在管理库中）的方式来指定服务器专用的配置项（比如数据库连接信息等）
- 执行服务器命令时需要关注环境变量，比如使用nvm来管理node版本的话，有可能导致PM2连接后找不到node（以及npm/pm2）所在路径，解决办法是在脚本最前面加上指定环境变量的脚本，例如`source ~/.bashrc`

## End

就是这样，水文一篇，你要是当成PM2的广告读也行，主要是这个功能真的是很方便。尤其是有多个环境的话，几条命令就能搞定，再也不用登录服务器手工做一堆事情了。

