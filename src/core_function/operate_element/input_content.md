# 输入内容

找到元素后，也会遇到需要输入内容的情况。

典型用法是：

```python
# 方式1：xpath的set_text方式
searchElementSelector = self.driver.xpath(locatorText)
searchElementSelector.set_text(text)
```

即可输入文字。
