# 查找元素

安卓测试期间，最常用的要属于，查找和定位页面中的相关元素了。

目前主要有2种方式去查找(定位）元素：

* 调用`driver`，直接传递(属性值对应的)参数
  * 返回值类型：`uiautomator2.session.UiObject`
    * 获取元素属性方式：`UiObject.info`
* 调用`driver.xpath`函数，传入xpath路径
  * 返回值类型：`uiautomator2.xpath.XPathSelector`
    * 获取元素属性方式：
      * 先：`XPathSelector.get()`得到`uiautomator2.xpath.XMLElement`
      * 再：`XMLElement.attrib`才能获取属性值
    * 注：
      * `xpath(xpathSelecotr).all()`得到是`XMLElement`的`list`列表，而不是`XPathSelector`
        * -> `XPathSelector`去`get()`或`all()`后，都是`XMLElement`
