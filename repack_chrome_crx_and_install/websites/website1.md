# website2
[Chrome 无法拖动安装crx扩展文件的解决办法](https://zhuanlan.zhihu.com/p/66229750)

 **安装离线的crx文件**
>目前Chrome 74不支持crx文件拖动安装了，解决办法很简单。
>
>1，将crx后缀改为rar，然后解压。
>
>2，`chrome://extensions`进入扩展程序界面，把解压好的文件拖过来进行了，就是从拖crx变成拖文件夹而已。

**将已安装的插件转为离线的crx文件**
>从谷歌商店安装好插件后，我们可以永久离线保存该插件。
>
>chrome://extensions，打包已经安装的文件。进入谷歌应用官方默认安装目录，那些英文开头的文件夹是无法打包，需要点开他们，找到他们数字版本命名的文件夹，打包即可。
>
>注:
>
>Chrome应用商店官方扩展的安装目录：
>
>`C:\Users\Bin\AppData\Local\Google\Chrome\User Data\Default\Extensions`
Chrome的安装目录：
>
>`C:\Users\Bin\AppData\Local\Google\Chrome\Application`
从Chrome官方应用商店安装的扩展，即安装在默认目录的扩展，在Chrome扩展管理中心删除后文件就删除了。
>
>而从其他文件夹拖过来的通过开发者模式手动安装的扩展，在Chrome扩展管理中心删除后文件依然存在。
>
编辑于 2019-12-28 16:06