# 输入内容

找到元素后，也会遇到需要输入内容的情况。

典型用法是：

```python
# 方式1：xpath的set_text方式
searchElementSelector = self.driver.xpath(locatorText)
searchElementSelector.set_text(text)
```

即可输入文字。

后记：已整理成独立函数：

```python
def selectorSetText(self, curXpathSelector, inputText):
    # Special: add click to try workaround for 360 pwd EditText input but input to 360 account EditText
    # curXpathSelector.click()
    # curXpathSelector.clear_text()
    selectorSetTextResp = curXpathSelector.set_text(inputText)
    logging.debug("selectorSetTextResp=%s", selectorSetTextResp) # selectorSetTextResp=None
    # 在set_text后，输入法会变成FastInputIME输入法
    # 用下面代码可以实现：关掉FastInputIME输入法，切换回系统默认输入法
    self.driver.set_fastinput_ime(False)
```

调用举例：

```python
Qihoo360_Account = "yourAccount"
accountXpath = """//android.widget.EditText[@resource-id="com.qihoo.gamecenter.pluginapk:id/lp_account_input_edit" and @index="1"]"""
accountSelector = self.driver.xpath(accountXpath)
self.selectorSetText(accountSelector, Qihoo360_Account)
```
