# 设备相关

此处整理出，和安卓设备相关的一些通用功能的函数和调用举例。

## 获取安卓设备信息

```python
def getDeviceInfo(self):
    return self.driver.device_info
```

调用：

```python
deviceInfo = self.getDeviceInfo()
```
## 获取安卓版本

```python
def getAndroidVersion(self):
    """返回安卓版本号，float值： 6.0，9.0 """
    deviceInfo = self.getDeviceInfo()
    logging.debug("deviceInfo=%s" % deviceInfo)
    androidVersionStr = deviceInfo["version"] # '6.0'
    androidVersionFloat = float(androidVersionStr)
    return androidVersionFloat
```

调用：

```python
    curAndroidVersionFloat = self.getAndroidVersion()
    ANDROID_VERSION_NEED_RESTART_U2 = 7.0
    if curAndroidVersionFloat <= ANDROID_VERSION_NEED_RESTART_U2:
        isNeedRestartU2 = True
```

## 获取安卓屏幕分辨率

```python
def getCurScreenResolution(self):
    """Get current screen resolution"""
    driverInfo = self.driver.info
    logging.debug("driverInfo=%s" % driverInfo)
    # displayWidth = driverInfo["displayWidth"]
    # displayHeight = driverInfo["displayHeight"]
    # logging.info("displayWidth=%s, displayHeight=%s", displayWidth, displayHeight)
    # deviceInfo = self.driver.device_info
    deviceInfo = self.getDeviceInfo()
    logging.debug("deviceInfo=%s" % deviceInfo)
    deviceDisplay = deviceInfo["display"]
    logging.debug("deviceDisplay=%s" % deviceDisplay)
    screenWidth = deviceDisplay["width"]
    screenHeight =  deviceDisplay["height"]
    logging.debug("screenWidth=%s, screenHeight=%s", screenWidth, screenHeight)
    if driverInfo["displayRotation"]:
        curScreenWidth = screenHeight
        curScreenHeight = screenWidth
    else:
        curScreenWidth = screenWidth
        curScreenHeight = screenHeight
    logging.debug("curScreenWidth=%s, curScreenHeight=%s", curScreenWidth, curScreenHeight)

    return (curScreenWidth, curScreenHeight)
```

调用：

```python
screenWidth, screenHeight = self.getCurScreenResolution()
```
