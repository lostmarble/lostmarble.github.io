

nodejs 有多个版本并行开发，最简单的方式是从官方网站下载已经编译好的安装包。网上有推荐安装nvm，然后使用nvm管理切换各个版本的nodejs，也是非常方便。
多个版本的[下载地址](https://nodejs.org/en/download/releases/)

可以需要的版本，下载解压到目录，如
```
~/workspace/nodejs/node-v0.12.9-linux-x64
```

然后执行
```bash
ln -s ~/workspace/nodejs/node-v0.12.9-linux-x64 node
```

这样方便以后版本切换
,我喜欢在~/.bashrc加载工作环境，在其加入最后一行
export PATH=$PATH:~/workspace/nodejs/node/bin
最后执行

```bash
. ~/.bashrc
```

这样node环境已经搭建好了

