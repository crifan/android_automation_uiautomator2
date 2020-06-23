# xpath

xpath本身是一套独立的技术，常用于web领域内。

此处uiautomator2也支持xpath，用于元素定位，可以实现复杂条件的元素的查找。

## xpath常见操作

### 定位节点和操作节点

```python
tbsNodeList = self.driver.xpath("//com.tencent.tbs.core.webkit.WebView").all()
```

### 获取属性content-desc的值

```python
eachTbsNode.attrib.get("content-desc", "")
```

### 给属性content-desc设置值

```python
eachTbsNode.attrib["content-desc"] = "add something to avoid NAF"
```

### 删除一个属性

```python
eachTbsNode.attrib.pop("NAF")
```

## 文档

关于Xpath的详细用法，见官网中的xpath的文档：

[uiautomator2/uiautomator2/ext/xpath at master · openatx/uiautomator2](https://github.com/openatx/uiautomator2/tree/master/uiautomator2/ext/xpath)

-》文档已经移至：

[uiautomator2/XPATH.md at master · openatx/uiautomator2](https://github.com/openatx/uiautomator2/blob/master/XPATH.md)


其内部用的lxml，具体功能和语法都可以参考：

[The lxml.etree Tutorial](https://lxml.de/tutorial.html)




