# 安卓自动化测试利器：uiautomator2

* 最新版本：`v2.3`
* 更新时间：`20210628`

## 简介

总结安卓设备自动化测试领域好用的库uiautomator2，包括简介；如何搭建环境；有哪些核心功能，比如监听、用xpath或driver传参等方式查找元素，且给出了具体的浏览器的输入框如何查找定位的例子、以及常见的操作元素，比如点击元素、输入内容等、如何获取当前屏幕的截图和xml源码；以及与u2相关的内容，比如辅助调试的weditor、adb、android-uiautomator-server、uiautomator；以及常见问题和经验，比如文字输入、NAF、long_click不工作、后台服务被杀掉等；以及一些源码分析；和通用代码段，包括工具类函数、adb相关、设备相关等；最后给出参考资料和文档。以及额外加上了很多实际案例，比如常见的确定类弹框按钮、自动关闭各大应用市场的广告类弹框、Vivo自动安装app、Vivo自动登录账号、奇虎360的自动登录账号。以及其他一些常见逻辑。

## 源码+浏览+下载

本书的各种源码、在线浏览地址、多种格式文件下载如下：

### Gitbook源码

* [crifan/android_automation_uiautomator2: 安卓自动化测试利器：uiautomator2](https://github.com/crifan/android_automation_uiautomator2)

#### 如何使用此Gitbook源码去生成发布为电子书

详见：[crifan/gitbook_template: demo how to use crifan gitbook template and demo](https://github.com/crifan/gitbook_template)

### 在线浏览

* [安卓自动化测试利器：uiautomator2 book.crifan.com](http://book.crifan.com/books/android_automation_uiautomator2/website)
* [安卓自动化测试利器：uiautomator2 crifan.github.io](https://crifan.github.io/android_automation_uiautomator2/website)

### 离线下载阅读

* [安卓自动化测试利器：uiautomator2 PDF](http://book.crifan.com/books/android_automation_uiautomator2/pdf/android_automation_uiautomator2.pdf)
* [安卓自动化测试利器：uiautomator2 ePub](http://book.crifan.com/books/android_automation_uiautomator2/epub/android_automation_uiautomator2.epub)
* [安卓自动化测试利器：uiautomator2 Mobi](http://book.crifan.com/books/android_automation_uiautomator2/mobi/android_automation_uiautomator2.mobi)

## 版权说明

此电子书教程的全部内容，如无特别说明，均为本人原创和整理。其中部分内容参考自网络，均已备注了出处。如有发现侵犯您版权，请通过邮箱联系我 `admin 艾特 crifan.com`，我会尽快删除。谢谢合作。

## 鸣谢

感谢我的老婆**陈雪**的包容理解和悉心照料，才使得我`crifan`有更多精力去专注技术专研和整理归纳出这些电子书和技术教程，特此鸣谢。

## 更多其他电子书

本人`crifan`还写了其他`100+`本电子书教程，感兴趣可移步至：

[crifan/crifan_ebook_readme: Crifan的电子书的使用说明](https://github.com/crifan/crifan_ebook_readme)
