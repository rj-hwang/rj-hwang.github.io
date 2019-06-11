# 绿化 NodeJS 开发环境 - Windows 篇

从 [NodeJS] 官方网站主页默认下载的是 `.msi` 扩展名的自动安装包，如 [node-v10.15.3-x64.msi]，下载后双击 `.msi` 文件按默认提示即可成功安装，这种情况下执行 `npm` 命令下载的依赖包默认缓存在当前用户目录下的子目录 `.npm` 下，如假设你登录的用户为 `{username}`，则在 Windows10 下对应的目录就是 `C:\Users\{username}\.npm`。

下面提供的是另外一种本人最喜欢的完全绿色的安装方式，不需要运行任何安装程序，下载官方 `.zip` 包，解压后配置相应的环境变量即可，同时我也将默认的 `.npm` 目录迁移到非系统分区的其它目录下，这至少有如下两大好处：

1. 日后升级 [NodeJS] 非常简单，删除旧版 `.zip` 包解压到的目录 ，重新下载最新版的官方 `.zip` 包解压到原来的位置即可。
2. 迁移 `.npm` 目录到非系统分区后，就算系统重装也不会影响原来已经下载缓存的依赖包，重新配置一下系统环境变量即可。

绿化 NodeJS 开发环境详细步骤如下：

1. 打开官方网站的下载页面 <https://nodejs.org/en/download>，按下图所示下载 64-bit 的 `Windows Binary (.zip)` 包，如当前最新的长期服务支持版本为 [node-v10.15.3-win-x64.zip] ：
    ![][nodejs-download-img]
2. 解压 `node-v10.15.3-win-x64.zip` 文件，解压后整理到目录 `D:\green\nodejs\node-latest-win-x64` 下，这个目录可以根据自己的需要自行定义，目录下文件结构应该类似如下：
    ```
    D:\green\nodejs\node-latest-win-x64
      ├ node_modules
      ├ node.exe
      ├ npm
      ├ npm.cmd
      ├ npx
      ├ npx.cmd
      ├ ...
    ```
    > 如果不是上述结构，请务必重新调整好。这里我将解压后默认的目录名称 `node-v10.15.3-win-x64` 更改为了 `node-latest-win-x64`，这样就可以在下次升级后，只要同样操作，就不需要重新修改下一步需要配置的系统环境变量 `Path` 的值。
3. 添加上述路径 `D:\green\nodejs\node-latest-win-x64` 到系统环境变量 `Path` 中，到此 `NodeJS` 就安装好可以使用了，下一步是将 `.npm` 缓存目录迁移到非系统分区。
4. 创建系统环境变量 `NPM_CONFIG_CACHE`，值设置为 `D:\data\.npm`，这个目录可以根据自己的需要自行定义，设置好后 `NodeJS` 的 `.npm` 缓存目录就会自动改为此目录了。
5. 验证安装配置的正确性：在命令行执行如下命令能看到版本号信息即代表全部配置成功，如下所示：
    ```
    C:\Users\rj>node -v
    v10.15.3

    C:\Users\rj>npm -v
    6.4.1
    ```


[NodeJS]: https://nodejs.org/en/download
[node-v10.15.3-x64.msi]: https://nodejs.org/dist/v10.15.3/node-v10.15.3-x64.msi
[node-v10.15.3-win-x64.zip]: https://nodejs.org/dist/v10.15.3/node-v10.15.3-win-x64.zip
[nodejs-download-img]: assets/20190319-1-nodejs-download.webp