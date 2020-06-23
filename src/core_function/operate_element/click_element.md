# 点击元素

找到元素后，往往涉及到点击元素。

## 举例：点击搜索按钮

此处，对于安卓手机，华为畅享6S DIG-AL00，当前微信的公众号搜索界面中，已经处于系统自带输入法：华为Skype输入法时

![android_input_method_search_button](../../assets/img/android_input_method_search_button.png)

用对应代码：

```python
self.driver.send_action("search")
```

可以实现点击对应的 蓝色搜索🔍按钮，触发搜索，进入搜索结果页面。

详见：

【已解决】uiautomator2中点击华为手机中系统自带Swype的输入法中的搜索按钮

