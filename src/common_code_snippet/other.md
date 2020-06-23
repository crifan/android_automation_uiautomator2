# 其他

## 坐标值

### boundsToCenterPoint：从bounds算出中间坐标值

```python
def boundsToCenterPoint(self, boundsStr):
    """
        从bounds转换出中间点位置坐标

        Example：
            bounds: '[156,1522][912,2027]'
            return: [534, 1774]
    """
    filterStr = re.sub('\[|,|\]', " ", boundsStr)
    boundStrList = filterStr.split()
    boundMap = map(int, boundStrList)
    boundIntList = list(boundMap)
    x0 = boundIntList[0]
    y0 = boundIntList[1]
    x1 = boundIntList[2]
    y1 = boundIntList[3]
    centerPoint = [(x1 + x0)//2,(y1 + y0)//2]
    return centerPoint
```

调用：

```python
centerPoint = self.boundsToCenterPoint(locatorBounds)
self.tap(centerPoint)
```

## xpath

### getCurPageSource：获取当前页面源码xml

```python
def getCurPageSource(self):
    # curPageSrcXml = self.driver.dump_hierarchy()
    curPageSrcXml = self.driver.dump_hierarchy(compressed=False, pretty=False)

    # output, exitCode = self.driver.shell(["adb", "shell", "uiautomator", "dump"])
    # output, exitCode = self.driver.shell(["uiautomator", "dump"])
    # output, exitCode = self.driver.shell("uiautomator dump")
    # output, exitCode = self.driver.shell(["shell", "uiautomator", "dump"])
    # curPageSrcXml = output

    return curPageSrcXml
```

调用：

```python
curPageSrcXml = self.getCurPageSource()
```

## 查找元素

### findAndClickNode：查找当前节点的父级符合条件的节点 并点击

```python
def findAndClickNode(self, curNodeXpath):
    """
        寻找可以clickable=true的当前或父级元素，并点击

        注：主要用于当节点clickable=false，点击无效时，使用此方法
    """
    foundAndClicked = False
    matchDict = {"clickable": "true"}
    clickableParentNode = self.findParentNode(curNodeXpath=curNodeXpath, matchDict=matchDict)
    if clickableParentNode:
        foundNodeAttrib = clickableParentNode.attrib
        clickableParentNode.click()
        foundAndClicked = True
        logging.info("clicked element [%s] found by [xpath=%s, match=%s]", foundNodeAttrib, curNodeXpath, matchDict)
    else:
        logging.warning("Fail click %s for not found %s(parent) node", curNodeXpath, matchDict)

    return foundAndClicked
```

调用：

```python
if curNodeXpath:
    foundAndClicked = self.findAndClickNode(curNodeXpath)
```

相关函数：

#### findParentNode：寻找父节点

```python
def findParentNode(self, curNodeXpath, matchDict, maxUpLevel=3):
    """
        寻找符合特定条件的父级节点，最多向上找3级

        如果当前节点符合条件，则返回当前节点
    """
    matchNode = None

    try:
        curNode = self.driver.xpath(curNodeXpath).get()
        curNodeAttrib = curNode.attrib # .attrib contain 'clickable'
        # curNodeInfo = curNode.info # .info not contain 'clickable'
        isCurMatch = self.isMatchNode(curNodeAttrib, matchDict)
        if isCurMatch:
            # current is match
            matchNode = curNode
        else:
            # try parent nodes
            curUpLevel = 1
            curParentNodeXpath = curNodeXpath
            while(curUpLevel <= maxUpLevel):
                curParentNodeXpath += "/.."
                curParentNode = self.driver.xpath(curParentNodeXpath).get()
                curParentNodeAttrib = curParentNode.attrib
                isCurParentMatch = self.isMatchNode(curParentNodeAttrib, matchDict)
                if isCurParentMatch:
                    matchNode = curParentNode
                    break

                curUpLevel += 1

    except XPathElementNotFoundError as xpathNotFoundErr:
        logging.error("XPathElementNotFoundError: %s", xpathNotFoundErr)

    if not matchNode:
        logging.warning("Not found match parent for xpath=%s and match=%s", curNodeXpath, matchDict)

    return matchNode
```

#### isMatchNode：节点是否匹配

```python
def isMatchNode(self, curNodeAttrib, toMathInfo):
    """判断当前节点属性是否满足条件"""
    isAllMatch = True
    for eachKey, eachToMatchValue in toMathInfo.items():
        if eachKey not in curNodeAttrib:
            isAllMatch = False
            break
    
        curValue = curNodeAttrib[eachKey]
        if curValue != eachToMatchValue:
            isAllMatch = False
            break

    return isAllMatch
```

#### findAndClickTextNode：寻找节点并点击

```python
    def findAndClickTextNode(self, text):
        """
            对于text类型节点：android.widget.TextView, text=xxx
            寻找可以clickable=true的当前或父级元素，并点击


            注：主要用于当text=xxx的节点clickable=false，点击无效时，使用此方法
        """
        curTextNodeXpath = "//android.widget.TextView[@text='%s']" % text
        self.findAndClickNode(curTextNodeXpath)
```

### xpathFindElement：用xpath查找元素

```python
def xpathFindElement(self, curClass=None, curId=None, curBounds=None):
    """
        find element by xpath

        return value type
            is: u2.xpath.XMLElement
            not: u2.session.UiObject
    """
    foundElement = None
    curXpath = self.generateElementXpath(curClass=curClass, curId=curId, curBounds=curBounds)
    try:
        foundElement = self.driver.xpath(curXpath).get()
    except XPathElementNotFoundError as xpathNotFoundErr:
        logging.error("XPathElementNotFoundError: %s from %s", xpathNotFoundErr, curXpath)

    return foundElement
```

调用：

（1）

```python
foundElement = self.xpathFindElement(curClass=locatorClass, curId=locatorId, curBounds=locatorBounds)
```

相关函数：

#### generateElementXpath：生成元素xpath

```python
def generateElementXpath(self, curClass=None, curId=None, curBounds=None):
    """generate element xpath"""
    # nodeXpath = ""
    # if locatorClass:
    #     nodeXpath =  "//%s[@bounds='%s']" % (locatorClass, locatorBounds) # "//android.widget.TextView[@bounds='[191,2060][430,2135]']"
    # elif locatorId:
    #     nodeXpath =  "//*[@resource-id='%s' and @bounds='%s']" % (locatorId, locatorBounds)
    # else:
    #     nodeXpath = "//*[@bounds='%s'" % locatorBounds

    classRule = "*"
    if curClass:
        classRule = curClass # 'android.widget.ImageView'

    propertyRule = ""
    if curId:
        propertyRule += "@resource-id='%s'" % curId
        # "@resource-id='com.netease.newsreader.activity:id/hs'"

    if curBounds:
        if propertyRule:
            propertyRule += " and "

        propertyRule += "@bounds='%s'" % curBounds
        # "@resource-id='com.netease.newsreader.activity:id/hs' and @bounds='[75,2098][141,2134]'"

    # TODO: add other support: text, desc, instance, ...
    curXpath = "//%s[%s]" % (classRule, propertyRule)
    # "//android.widget.ImageView[@resource-id='com.netease.newsreader.activity:id/hs' and @bounds='[75,2098][141,2134]']"

    return curXpath
```

调用：

```python
curClassname = None
curResId = None
curBoundsStr = None

# curAttrib = foundElement.attrib
# AttributeError: 'UiObject' object has no attribute 'attrib'
if hasattr(foundElement, "attrib"):
    curAttrib = foundElement.attrib
    # {'index': '0', 'text': '', 'resource-id': 'com.netease.newsreader.activity:id/hs', 'package': 'com.netease.newsreader.activity', 'content-desc': '', 'checkable': 'false', 'checked': 'false', 'clickable': 'false', 'enabled': 'true', 'focusable': 'false', 'focused': 'false', 'scrollable': 'false', 'long-clickable': 'false', 'password': 'false', 'selected': 'true', 'visible-to-user': 'true', 'bounds': '[75,2098][141,2134]'}
    curResId = curAttrib["resource-id"]
    curBoundsStr = curAttrib["bounds"]
else:
    # # for debug
    # self.debugPrintElement(foundElement, "no attrib")
    logging.debug("")

curInfo = foundElement.info
# {'bounds': {'bottom': 2134, 'left': 75, 'right': 141, 'top': 2098}, 'className': 'android.widget.ImageView', 'contentDescription': '', 'enabled': 'true', 'focusable': 'false', 'focused': 'false', 'longClickable': 'false', 'packageName': 'com.netease.newsrea....activity', 'scrollable': 'false', 'selected': 'true', 'text': ''}
if not curClassname:
    curClassname = curInfo["className"] # 'android.widget.ImageView'

if not curBoundsStr:
    boundsDict = curInfo["bounds"]
    x0 = boundsDict["left"]
    y0 = boundsDict["top"]
    x1 = boundsDict["right"]
    y1 = boundsDict["bottom"]
    curBoundsStr = "[%d,%d][%d,%d]" % (x0, y0, x1, y1)
    # '[75,2098][141,2134]'

if not curResId:
    if "resourceName" in curInfo:
        curResId = curInfo["resourceName"] # 'com.netease.newsreader.activity:id/bn5'

curNodeXpath = self.generateElementXpath(
    curClass=curClassname,
    curId=curResId,
    curBounds=curBoundsStr,
)
```

