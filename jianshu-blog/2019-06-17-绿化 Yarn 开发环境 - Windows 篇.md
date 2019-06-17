# 绿化 Yarn 开发环境 - Windows 篇

由于 [Yarn] 使用缓存和并发来处理的原因，[Yarn] 比 [NodeJS] 快的可不是一点点，开发 [NodeJS] 应用一定要用 [Yarn]。

[Yarn] 的官方安装包是直接从 GitHub 下载的，下载地址为 <https://github.com/yarnpkg/yarn/releases>，打开网页后点击相应的版本号即可进入文件下载页面，如 `1.16.0` 稳定版文件下载地址为 <https://github.com/yarnpkg/yarn/releases/tag/v1.16.0>，如下图所示，点击 [yarn-v1.16.0.tar.gz] 就可下载绿色版的安装包：

![][yarn-download-img]

> 如果下载的是 `.msi` 扩展名的自动安装包，如 [yarn-v1.16.0.msi]，下载后双击 `.msi` 文件按默认提示即可成功安装，这种情况下执行 `yarn` 命令下载的依赖包默认缓存在当前用户目录下的子目录 `.yarn` 下，如假设你登录的用户为 `{username}`，则在 Windows10 下对应的目录就是 `C:\Users\{username}\.yarn`。

下面提供的是另外一种本人最喜欢的完全绿色的安装方式，不需要运行任何安装程序，下载官方 `.tar.gz` 包，解压后配置相应的环境变量即可，同时我也将默认的 `.yarn` 目录迁移到非系统分区的其它目录下，这至少有如下两大好处：

1. 日后升级 [Yarn] 非常简单，删除旧版 `.tar.gz` 包解压到的目录 ，重新下载最新版的官方 `.tar.gz` 包解压到原来的位置即可。
2. 迁移 `.yarn` 目录到非系统分区后，就算系统重装也不会影响原来已经下载缓存的依赖包，重新配置一下系统环境变量即可。

绿化 [Yarn] 开发环境详细步骤如下：

1. 解压 `yarn-v1.16.0.tar.gz` 文件，解压后整理到目录 `D:\green\yarn\yarn-latest` 下，这个目录可以根据自己的需要自行定义，目录下的文件结构应该如下：
    ```
    D:\green\yarn\yarn-latest
      ├ bin
      │ ├ yarn
      │ ├ yarn.cmd
      │ ├ yarn.js
      │ ├ yarnpkg
      │ └ yarnpkg.cmd
      ├ lib
      │ ├ cli
      │ └ v8-compile-cache.js
      ├ LICENSE
      ├ package.json
      └ README.md
    ```
    > 如果不是上述结构，请务必重新调整好。这里我将解压后默认的目录名称 `yarn-v1.16.0.` 更改为了 `yarn-latest`，这样就可以在下次升级后，只要同样操作，就不需要重新修改下一步需要配置的系统环境变量 `Path` 的值。
3. 添加上述路径 `D:\green\yarn\yarn-latest\bin` 到系统环境变量 `Path` 中，到此 `Yarn` 就安装好可以使用了，下一步是将 `.yarn` 缓存目录迁移到非系统分区。
4. 创建系统环境变量 `YARN_CACHE_FOLDER`，值设置为 `D:\data\.yarn`，这个目录可以根据自己的需要自行定义，设置好后 `Yarn` 的 `.yarn` 缓存目录就会自动改为此目录了。
5. 验证安装配置的正确性：在命令行执行如下命令能看到版本号信息即代表全部配置成功，如下所示：
    ```
    C:\Users\rj>yarn -v
    1.16.0
    ```


[Yarn]: https://yarnpkg.com
[NodeJS]: https://nodejs.org
[yarn-download-img]: assets/20190617-download-yarn-1.16.0.webp
[yarn-v1.16.0.tar.gz]: https://github.com/yarnpkg/yarn/releases/download/v1.16.0/yarn-v1.16.0.tar.gz
[yarn-v1.16.0.msi]: https://github.com/yarnpkg/yarn/releases/download/v1.16.0/yarn-1.16.0.msi