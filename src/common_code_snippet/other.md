# 其他

## 获取元素属性

```python
# 获取元素属性值
def get_ElementBounds(self, element):
    # 将元素坐标转成数组
    if self.isAndroid:
        # <node index="1" text="" resource-id="com.tencent.mm:id/l1"  。。。 bounds="[0,48][720,134]">
        bounds = element.attrib.get("bounds")
        return list(map(int, re.sub('\[|,|\]', " ", bounds).split())) if bounds else [0, 0, 0, 0]
    elif self.isiOS:
        # <XCUIElementTypeButton 。。。 name="返回" label="返回" 。。。 x="20" y="20" width="30" height="44"/>
        # attrib = element.attrib
        # xStr = attrib["x"]
        # yStr = attrib["y"]
        # widthStr = attrib["width"]
        # heightStr = attrib["height"]
        # x = int(xStr)
        # y = int(yStr)
        # width = int(widthStr)
        # height = int(heightStr)
        x = self.get_ElementX(element)
        y = self.get_ElementY(element)
        width = self.get_ElementWidth(element)
        height = self.get_ElementHeight(element)
        x1 = x + width
        y1 = y + height
        boundList = [x, y, x1, y1]
        return boundList # [16, 20, 46, 64]

def get_ElementX(self, element):
    if self.isAndroid:
        bounds = self.get_ElementBounds(element)
        x = bounds[0]
        return x
    elif self.isiOS:
        attrib = element.attrib
        xStr = attrib["x"]
        x = int(xStr)
        return x

def get_ElementY(self, element):
    if self.isAndroid:
        bounds = self.get_ElementBounds(element)
        y = bounds[1]
        return y
    elif self.isiOS:
        attrib = element.attrib
        yStr = attrib["y"]
        y = int(yStr)
        return y

def get_ElementWidth(self, element):
    if self.isAndroid:
        bounds = self.get_ElementBounds(element)
        width = bounds[2] - bounds[0]
        return width
    elif self.isiOS:
        attrib = element.attrib
        widthStr = attrib["width"]
        width = int(widthStr)
        return width

def get_ElementHeight(self, element):
    if self.isAndroid:
        bounds = self.get_ElementBounds(element)
        height = bounds[3] - bounds[1]
        return height
    elif self.isiOS:
        attrib = element.attrib
        heightStr = attrib["height"]
        height = int(heightStr)
        return height

def get_ElementSize(self, element):
    # 获取元素方框大小
    bounds = self.get_ElementBounds(element)
    return (bounds[2] - bounds[0]) * (bounds[3] - bounds[1])

def get_ElementPoint(self,element):
    # 获取元素中心点坐标
    bounds = self.get_ElementBounds(element)
    return [(bounds[2] + bounds[0])//2,(bounds[3] + bounds[1])//2]

def get_ElementText(self, element):
    if self.isAndroid:
        # 返回元素text文本
        textKey = "text"
    elif self.isiOS:
        # 返回元素label
        textKey = "label"
    textValue = element.attrib.get(textKey, "")
    return textValue

def get_ElementContentdesc(self, element):
    if self.isAndroid:
        # 返回元素content-desc文本
        descKey = "content-desc"
    elif self.isiOS:
        # 返回元素value
        descKey = "value"
    descValue = element.attrib.get(descKey, "")
    return descValue

def get_ElementDescribe(self, element):
    # # 返回元素text文本和content-desc文本
    # 返回元素 文本 和 描述
    elementText = self.get_ElementText(element)
    elementContentDesc = self.get_ElementContentdesc(element)
    descText = elementText + elementContentDesc
    return descText
```

## 判断是否是布局类型的元素

```python
def is_element_layout_Android(self, element):
    # 判断元素是否是out类型(如LinearLayout、RelativeLayout)
    # return "Layout" in element.attrib.get("class")
    curClass = element.attrib.get("class")
    #TODO: 换成re正则匹配 xxxLayout ？
    isLayout = "Layout" in curClass
    # 可能：
    # 	android.widget.FrameLayout
    #	android.widget.LinearLayout
    # 	android.widget.RelativeLayout
    # for debug
    if isLayout:
        knownLayoutList = [
            "android.widget.FrameLayout",
            "android.widget.LinearLayout",
            "android.widget.RelativeLayout",
        ]
        foundNew = curClass not in knownLayoutList
        if foundNew:
            print("curClass=%s" % curClass)
    return isLayout
```

## 判断元素是否是某种类型

```python
def is_element_Button(self, element):
    # 元素是否为Button
    if self.isAndroid:
        # return "Button" in element.attrib.get("class")
        return self.is_element_SomeType_Android(element, "Button")
    elif self.isiOS:
        # <XCUIElementTypeButton type="XCUIElementTypeButton" name="返回" label="返回" enabled="true" visible="true" x="20" y="20" width="30" height="44"/>
        # iOSTagButton = "XCUIElementTypeButton"
        # elementTag = element.tag
        # isButton = elementTag == iOSTagButton
        # return isButton
        return self.is_element_SomeType_iOS(element, "XCUIElementTypeButton")

def is_element_Image(self, element):
    # 元素是否为ImageView
    if self.isAndroid:
        # return "Image" in element.attrib.get("class")
        return self.is_element_SomeType_Android(element, "Image")
    elif self.isiOS:
        # <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="0" y="64" width="414" height="56"/>
        # iOSTagImage = "XCUIElementTypeImage"
        # elementTag = element.tag
        # isImage = elementTag == iOSTagImage
        # return isImage
        return self.is_element_SomeType_iOS(element, "XCUIElementTypeImage")

def is_element_EditText(self, element):
    if self.isAndroid:
        # 元素是否为EditText
        # return "EditText" in element.attrib.get("class")
        return self.is_element_SomeType_Android(element, "EditText")
    elif self.isiOS:
        # <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="动卡空间" name="动卡空间" label="动卡空间" enabled="true" visible="true" x="68" y="395" width="338" height="26"/>
        # iOSTagStaticText = "XCUIElementTypeStaticText"
        # elementTag = element.tag
        # isStaticText = elementTag == iOSTagStaticText
        # return isStaticText
        # return self.is_element_SomeType_iOS(element, "XCUIElementTypeStaticText")

        # <XCUIElementTypeSearchField type="XCUIElementTypeSearchField" value="gh_cfcfcee032cc" name="搜索" label="搜索" enabled="true" visible="true" x="41" y="27" width="319" height="36">
        #	<XCUIElementTypeButton type="XCUIElementTypeButton" name="清除文本" label="清除文本" enabled="true" visible="false" x="335" y="35" width="20" height="20"/>
        # </XCUIElementTypeSearchField>
        isSearchField = self.is_element_SomeType_iOS(element, "XCUIElementTypeSearchField")
        # <XCUIElementTypeTextField type="XCUIElementTypeTextField" value="请输入手机号" label="" enabled="true" visible="true" x="107" y="188" width="265" height="24"/>
        isTextField = self.is_element_SomeType_iOS(element, "XCUIElementTypeTextField")
        # 
        isSecureTextField = self.is_element_SomeType_iOS(element, "XCUIElementTypeSecureTextField")
        isEditableText = isSearchField or isTextField or isSecureTextField
        return isEditableText

def is_element_Link(self, element):
    # 元素是否是 XCUIElementTypeLink
    """
        <XCUIElementTypeLink type="XCUIElementTypeLink" name="阅读原文" label="阅读原文" enabled="true" visible="false" x="16" y="1512" width="64" height="19">
            <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="阅读原文" name="阅读原文" label="阅读原文" enabled="true" visible="false" x="16" y="1512" width="64" height="19"/>
        </XCUIElementTypeLink>
    """
    return self.is_element_SomeType_iOS(element, "XCUIElementTypeLink")

def is_element_SomeType_iOS(self, element, typeName):
    elementType = None
    if hasattr(element, "tag"):
        # lxml Element
        elementTag = element.tag
        elementType = elementTag
    elif hasattr(element, "attrs"):
        # BeautifulSoup soup node
        elementAttrDict = element.attrs
        elementType = elementAttrDict.get("type")
    isCurrentType = elementType == typeName
    return isCurrentType

def is_element_SomeType_Android(self, element, typeName):
    curClass = element.attrib.get("class")
    isTypeInClass = typeName in curClass
    isCurrentType = isTypeInClass
    return isCurrentType
```

## 点击元素（中间坐标值）

```python
def clickElementCenterPosition(self, curElement):
    """Click center position of element

    Args:
        curElement (Element): Beautiful soup / lxml element / wda Element
    Returns:
        bool
    Raises:
    """
    hasClicked = False
    # centerPos = None
    centerX = None
    centerY = None

    hasBounds = hasattr(curElement, "bounds")
    curBounds = None
    if hasBounds:
        curBounds = curElement.bounds

    if hasBounds and curBounds:
        # wda element
        if hasattr(curBounds, "center"):
            # is wda Rect
            curRect = curBounds
            rectCenter = curRect.center
            centerX = rectCenter[0]
            centerY = rectCenter[1]
    else:
        attrDict = None
        if hasattr(curElement, "attrs"):
            # Beautiful soup node
            attrDict = curElement.attrs
        elif hasattr(curElement, "attrib"):
            # lxml element
            attrDict = dict(curElement.attrib)

        if attrDict:
            logging.info("attrDict=%s", attrDict)
            hasCoordinate = ("x" in attrDict) and ("y" in attrDict) and ("width" in attrDict) and ("height" in attrDict)
            if hasCoordinate:
                x = int(attrDict["x"])
                y = int(attrDict["y"])
                width = int(attrDict["width"])
                height = int(attrDict["height"])
                centerX = x + int(width / 2)
                centerY = y + int(height / 2)

    if centerX and centerY:
        centerPos = (centerX, centerY)
        self.tap(centerPos)
        logging.info("Clicked center position: %s", centerPos)
        hasClicked = True

    return hasClicked
```

调用：

```python
moreInfoSoup = parentCellSoup.find(
    'XCUIElementTypeButton',
    attrs={"type": "XCUIElementTypeButton", "name": "更多信息", "enabled":"true", "visible":"true"},
)
if moreInfoSoup:
    clickedOk = self.clickElementCenterPosition(moreInfoSoup)
```

或：

```python
page = self.get_page_source()
backElement, newPage = self.findRealBackElement(page)
if backElement is not None:
    isFoundAndClicked = self.clickElementCenterPosition(backElement)
```

或：

```python
# try return to main page, by find main menu and click first main menu
mainMenuList = self.get_elements_MainMenu(page)
if mainMenuList:
    firstMainMenu = mainMenuList[0]
    clickOk = self.clickElementCenterPosition(firstMainMenu)
```

或：

```python
isGetProxyTypeOk, respInfo = self.iOSLaunchSettingsAndGetProxyType()
curProxySoup = respInfo
curProxyAttrDict = curProxySoup.attrs
curTypeName = curProxyAttrDict.get("value")

# into config proxy page
self.clickElementCenterPosition(curProxySoup)
```

## 电脑相关

### 获取电脑序列号

```python
def getSerialNumber(self):
    """get current computer serial number"""
    # cmd = "wmic bios get serialnumber"
    cmd = ""
    if CommonUtils.osIsWinows():
        # Windows
        cmd = "wmic bios get serialnumber"
    elif CommonUtils.osIsMacOS():
        # macOS
        cmd = "system_profiler SPHardwareDataType | awk '/Serial/ {print $4}'"
    # TODO: add support other OS
    # AIX: aix
    # Linux: linux
    # Windows/Cygwin: cygwin

    serialNumber = ""
    lines = CommonUtils.get_cmd_lines(cmd)
    if CommonUtils.osIsWinows():
        # Windows
        serialNumber = lines[1]
    elif CommonUtils.osIsMacOS():
        # macOS
        serialNumber = lines[0] # C02Y3N10JHC8, 'VMfvNykazWi1'

    return serialNumber
```

调用：

```python
serialNumber = self.getSerialNumber() # 'VMfvNykazWi1'
```


## 调试相关

在安卓手机测试期间，往往会遇到一些和调试相关内容，此处整理出其中相对通用部分，供参考。

### 缩放图片（到原始尺寸比例）

```python
def scaleToOrginSize(self, screenshotImgPath, curScale):
    """resize to original screen size, according to session scale"""
    curScreenImg = Image.open(screenshotImgPath)
    originSize = curScreenImg.size # 750x1334
    newWidthInt = int(float(originSize[0]) / curScale)
    newHeightInt = int(float(originSize[1]) / curScale)
    scaledSize = (newWidthInt, newHeightInt) # 375x667
    scaledFile = screenshotImgPath
    CommonUtils.resizeImage(curScreenImg, newSize=scaledSize, outputImageFile=scaledFile)
    return scaledFile
```

### 获取当前屏幕截图文件

```python
def getCurScreenshot(self, saveFolder=None):
    """get current screenshot image file path"""

    curDatetimeStr = CommonUtils.getCurDatetimeStr() # '20200422_144915'
    # suffix = "png"
    suffix = "jpg" # '20200422_144915.jpg'
    curFilename = "%s.%s" % (curDatetimeStr, suffix)
    if not saveFolder:
        if self.isAndroid:
            # saveFolder = self.config["CurAndroidAppScreenshotRoot"]
            # saveFolder = self.config["CurAndroidWeixinScreenshotRoot"]
            # saveFolder = self.config["debug"]["screenshot"]["Android"]["weixin"]
            saveFolder = self.config["debug"]["screenshot"]["Android"]["gameApp"] # 'debug/Android/app/游戏app/screenshot'
        elif self.isiOS:
            # saveFolder = self.config["CuriOSWeixinScreenshotRoot"]
            # saveFolder = self.config["CuriOSAppPageSourceRoot"]
            # saveFolder = self.config["debug"]["pageSource"]["iOS"]["app"]
            # saveFolder = self.config["debug"]["screenshot"]["iOS"]["weixin"]
            # saveFolder = self.config["debug"]["screenshot"]["iOS"]["app"]
            saveFolder = self.config["debug"]["screenshot"]["iOS"]["normalApp"]
    # add current date sub folder
    curDateStr = CommonUtils.getCurDatetimeStr("%Y%m%d") # '20210107'
    saveFolder = os.path.join(saveFolder, curDateStr) # 'debug/Android/app/游戏app/screenshot/20210107'
    CommonUtils.createFolder(saveFolder)
    fullImgFilePath = os.path.join(saveFolder, curFilename)
    beforeDriverSceenshotTime = datetime.now()
    if self.isAndroid:
        fullImgFilePath = self.driver.screenshot(fullImgFilePath) # 'debug/Android/app/游戏app/screenshot/20201208_205117.jpg'
        # optimize size
        displayInfo = self.driver.device_info["display"] # {'width': 720, 'height': 1600}
        originSize = (displayInfo["height"], displayInfo["width"]) # (1600, 720)
        CommonUtils.resizeImage(fullImgFilePath, originSize, outputImageFile=fullImgFilePath)
    elif self.isiOS:
        fullImgFilePath = self.debugiOSSaveScreenshot(saveFolder=saveFolder, curScale=self.curSession.scale)
    afterDriverSceenshotTime = datetime.now()
    driverSceenshotTime = afterDriverSceenshotTime - beforeDriverSceenshotTime
    logging.debug("driver screenshot time: %s", driverSceenshotTime)
    return fullImgFilePath
```


### 给当前屏幕截图加标记（红框）

```python
def debugDrawScreenRect(self, curRect, curImgPath=None, isShow=False, isAutoSave=True, isDrawClickedPosCircle=False):
    """for debug, draw rectange for current screenshot"""
    if not curImgPath:
        curImgPath = self.getCurScreenshot()

    curImg = CommonUtils.imageDrawRectangle(
        curImgPath,
        curRect,
        isShow=isShow,
        isAutoSave=isAutoSave,
        isDrawClickedPosCircle=isDrawClickedPosCircle,
    )

    return curImg
```

### 给元素加边框标记

```python
def debugDrawElementRect(self, elementList, curImgPath=None, isShowEach=False, isSaveEach=True, isDrawInSinglePic=False):
    """for debug, to draw rectange for each element in current screenshot"""
    if not curImgPath:
        curImgPath = self.getCurScreenshot()

    curImg = Image.open(curImgPath)

    for eachElement in elementList:
        curBoundList = self.get_ElementBounds(eachElement)
        curWidth = curBoundList[2] - curBoundList[0]
        curHeight = curBoundList[3] - curBoundList[1]
        curRect = [curBoundList[0], curBoundList[1], curWidth, curHeight]
        curTimeStr = CommonUtils.getCurDatetimeStr("%H%M%S")
        curSaveTal = "_rect_{}_%x|%y|%w|%h".format(curTimeStr) # '_rect_155618_%x|%y|%w|%h'
        curInputImg = None
        if isDrawInSinglePic:
            curInputImg = curImg
        else:
            curInputImg = curImgPath
        curImg = CommonUtils.imageDrawRectangle(
            curInputImg,
            curRect,
            isShow=isShowEach,
            isAutoSave=isSaveEach,
            saveTail=curSaveTal,
            isDrawClickedPosCircle=False,
        )

    # always save final result
    curTimeStr = CommonUtils.getCurDatetimeStr("%H%M%S")
    finalSaveTail = "_rect_all_%s" % curTimeStr
    imgFolderAndName, pointSuffix = os.path.splitext(curImgPath)
    imgFolderAndName = imgFolderAndName + finalSaveTail
    finalImgPath = imgFolderAndName + pointSuffix
    curImg.save(finalImgPath)

    return
```

### 保存当前截图对应的xml源码

```python
def debugSaveCurPageSource(self, filePrefix="", saveFolder=None):
    """for debug, save current page source xml file"""
    savedSourceFile = None
    curDatetimeStr = CommonUtils.getCurDatetimeStr()
    sourceFormat="xml"
    # sourceFilename = "%s_source.%s" % (curDatetimeStr, sourceFormat) # '20200221_152817_source.xml'
    sourceFilename = "%s.%s" % (curDatetimeStr, sourceFormat)
    if filePrefix:
        sourceFilename = "%s_%s" % (filePrefix, sourceFilename)
        # 'com.netease.cloudmusic_20200221_170337.xml'
    
    if not saveFolder:
        # if self.isAndroid:
        # 	# saveFolder = self.config["CurAndroidAppPageSourceRoot"]
        # 	# saveFolder = self.config["CurAndroidWeixinPageSourceRoot"]
        # 	# saveFolder = self.config["debug"]["pageSource"]["Android"]["weixin"]
        # 	saveFolder = self.config["debug"]["pageSource"]["Android"]["app"]
        # elif self.isiOS:
        # 	# saveFolder = self.config["CuriOSWeixinPageSourceRoot"]
        # 	# saveFolder = self.config["debug"]["pageSource"]["iOS"]["weixin"]
        # 	saveFolder = self.config["debug"]["pageSource"]["iOS"]["app"]

        # if self.isAndroid:
        # 	platformType = "Android"
        # elif self.isiOS:
        # 	platformType = "iOS"
        # taskType = self.taskType
        # saveFolder = self.config["debug"]["pageSource"][platformType][taskType]
        saveFolder = self.config["debug"]["pageSource"][self.platformType][self.taskType]

    CommonUtils.createFolder(saveFolder)
    sourceFilename = os.path.join(saveFolder, sourceFilename)

    pageSource = self.getCurPageSource()
    CommonUtils.saveTextToFile(sourceFilename, pageSource)
    savedSourceFile = sourceFilename
    logging.debug("saved page source: %s", savedSourceFile)
    return savedSourceFile
```

### 保存当前屏幕的图片和源码

```python
def debugSaveScreenAndSource(self):
    self.getCurScreenshot()
    self.debugSaveCurPageSource()
```
### 打印元素属性值

```python
def debugPrintElement(self, curElement, prefix=""):
    """for debug, to print current element"""
    curElementStr = ""
    curInfoDict = {}
    keyList = []
    if self.isAndroid:
        if hasattr(curElement, "attrib"):
            curInfoDict = curElement.attrib
            keyList = ["resource-id", "class", "bounds", "text"]
        else:
            curInfoDict = curElement.info
            keyList = ["resourceName", "className", "bounds", "text"]
    elif self.isiOS:
        curInfoDict = curElement.attrib
        # keyList = ["type", "name", "label", "value", "enabled", "visible"]
        keyList = ["type", "name", "label", "value", "enabled", "visible", "x", "y", "width", "height"]

    valueList = []
    for eachKey in keyList:
        if eachKey in curInfoDict.keys():
            eachValue = curInfoDict.get(eachKey)
            eachValueStr = str(eachValue)
            valueList.append(eachValueStr)
        # else:
        # 	logging.debug("no %s key for %s", eachKey, curInfoDict)

    curElementStr = " | ".join(valueList)
    logging.info("%s element: %s", prefix, curElementStr)
    return
```

调用：

```python
self.debugPrintElement(curSubElement, "is subSubLen=1")
```

