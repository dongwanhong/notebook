# Nvm 使用笔记
之前通过 `Homebre` 安装 `nvm` 时，然后通过 `nvm` 来执行 `nvm ls-remote` 查询时始终返回 N/A，网上的解决方法提示修改镜像源：`export NVM_NODEJS_ORG_MIRROR=http://nodejs.org/dist`，后来我发现是因为我修改了 ss 服务，直接取消修改就好咯。

安装：

```bash
$ brew install nvm
```

查看安装的版本：

```bash
$ nvm --version
```

查看 `nvm` 的帮助信息：

```bash
$ nvm --help
```

查看可以安装的版本：

```bash
$ nvm ls-remote
```

可以查看的LTS版本：

```bash
$ nvm ls-remote --lts
```

安装指定版本的 `node`：

```bash
$ nvm install <version>
```

用 `nvm` 给不同的版本号设置别名(默认最新版的别名为 node)：

```bash
$ nvm alias <awesome_name> <version>
```

取消别名：

```bash
$ nvm unalias <awesome_name>
```

查看已经安装的版本：

```bash
$ nvm ls
```

切换使用的版本：

```bash
$ nvm use <version>
```

查看当前版本：

```bash
$ nvm current
```

卸载制定版本的 `node`：

```bash
$ nvm uninstall <version>
```

## 参考链接
 * [http://bubkoo.com/2017/01/08/quick-tip-multiple-versions-node-nvm/](http://bubkoo.com/2017/01/08/quick-tip-multiple-versions-node-nvm/)
 * [https://www.imooc.com/article/17674](https://www.imooc.com/article/17674)
 * [https://github.com/creationix/nvm](https://github.com/creationix/nvm)