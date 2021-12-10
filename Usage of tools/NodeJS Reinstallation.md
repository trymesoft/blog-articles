Posted on 2019-01-05
> 最近使用 Node.js 构建一些东西，但是环境被我搞坏了，各种依赖全部出问题了，只好删除重新配置环境，将 <code>Node.js</code> 和 <code>npm</code> 全部删除。

### 完全移除 Node.js 及 npm
由于是 macOS 系统，以为直接从应用程序删除即可，结果发现 Node.js 是当时配置安装的（十分后悔为什么当时没用 brew 安装?），Google 了一下，发先完全删除 Node.js 还真鸡儿麻烦，所有的都得手动删除，不过 幸好 Node.js 的目录都很好识别。

- 删除 <code>/usr/local</code> 下 <code>lib</code>、<code>include</code>、<code>bin</code> 下所有 <code>node</code> 开头的文件/文件夹
- 删除用户目录下的 <code>.npmrc</code> 、<code>.npm</code>、<code>.node-gyp</code>、<code>.node_repl_history</code> 等文件/文件夹
- 删除 <code>/usr/local/share/man/man1/、/usr/local/lib/dtrace/、/opt/local/bin/、
/opt/local/include/、/opt/local/lib/、/usr/local/share/doc/</code> 下所有 <code>node*</code> 文件/文件夹

**上述文件/文件夹视个人情况而定。**

PS：如果是 brew 安装的，直接 <code>brew uninstall node</code> 即可。

------
### 使用 Homebrew 安装 Node.js

引用 Homebrew 官网介绍：<code>Homebrew The missing package manager for macOS</code>

如果没有安装 Homebrew，可以参考 [https://brew.sh/](https://brew.sh/) 安装。

下面使用 brew 命令安装 Node.js，直接运行 `brew install node`，然后默默等待即可，安装完成后可以执行 `node -v` 和 `npm -v` 查看 Node.js 及 npm 版本。

PS：在安装完成后使用的时候遇到了一个问题，因为印象中貌似有一次使用 `npm` 安装东西的时候遇到了一个权限问题，初次重装模块时，为了避免权限问题，我使用了管理员权限安装 `sudo npm i hexo-cli -g` ，结果遇到了一个大问题，报错大致如下：

```shell
...
gyp ERR! clean error 
gyp ERR! stack Error: EACCES: permission denied, rmdir 'build'
gyp ERR! stack     at Error (native)
gyp ERR! System Darwin 18.2.0
gyp ERR! command
...
```

**解决：** 执行 `sudo chown -R $(whoami):staff /usr/local/lib/node_modules` 授予当前用户权限即可，然后万万不能将 <code>sudo</code> 和 <code>npm</code> 一起用。[参考](https://github.com/ionic-team/ionic-cli/issues/3127)

