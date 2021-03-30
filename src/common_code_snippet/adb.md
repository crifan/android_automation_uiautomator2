# adb

uiautomator2操作安卓设备期间，往往会涉及到，借助于安卓体系内本身就有的工具 `adb`，去实现对设备的一些操控。

此处整理出一些常见的用法和通用功能。

## 获取当前安卓手机名

```python
def get_phone_name_Android(self):
    # cmd = 'adb -s {} shell getprop ro.product.model'.format(self.device)
    cmd = 'adb -s {} shell getprop ro.product.name'.format(self.device)
    text = CommonUtils.get_cmd_lines(cmd,text=True)
    # https://miuiver.com/xiaomi-device-codename/
    #     begonia -> 红米Note 8内部代号为 "begonia"
    return re.sub("\s","",text)
    # isRunCmdOk, outputText = self.getCommandOutput(cmd)
    # if isRunCmdOk:
    #     phoneName = outputText
    # else:
    #     phoneName = ""
    # return phoneName
```

调用：

```python
def get_phone_name(self):
    # 获取手机名称，以提取配置信息
    if self.isAndroid:
        return self.get_phone_name_Android()
```

## 获取当前连接的设备

```python
def get_devices_Android(self):
    lines = CommonUtils.get_cmd_lines('adb devices')
    return [line.split()[0] for line in lines if line and not "devices" in line]
```

调用：

```python
devices = self.get_devices_Android()
```

相关命令输出举例：

```bash
➜  ~ adb devices
List of devices attached
8c8a4d4d    device
```

### 优化版：获取安卓设备列表

```python
def getAndroidDeviceList(self, isGetDetail=False):
    """Get android device list

    Args:
        isGetDetail (bool): True to use `adb devices -l`, False to use `adb devices`
    Returns:
        device list(list)
    Raises:
    Examples:
        output: 
            False -> ['2e2a0cb1', 'orga4pmzee4ts47t', '192.168.31.84:5555']
            True -> [{'2e2a0cb1': {'usb': '338952192X', 'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '4'}}, {'orga4pmzee4ts47t': {'usb': '338886656X', 'product': 'atom', 'model': 'M2004J7AC', 'device': 'atom', 'transport_id': '24'}}, {'192.168.31.84:5555': {'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '5'}}]
    """
    deviceList = []

    getDevicesCmd = 'adb devices'
    if isGetDetail:
        getDevicesCmd += " -l"
    logging.debug("getDevicesCmd=%s", getDevicesCmd)

    deviceLines = CommonUtils.get_cmd_lines(getDevicesCmd)
    logging.debug("deviceLines=%s", deviceLines)
    # ['List of devices attached', '2e2a0cb1\tdevice', 'orga4pmzee4ts47t\tdevice', '192.168.31.84:5555\tdevice', '']
    """
    adb devices :
        List of devices attached
        2e2a0cb1	device
        orga4pmzee4ts47t	device
        192.168.31.84:5555	device
    """

    """
    adb devices -l:
        List of devices attached
        2e2a0cb1               device usb:338952192X product:PD2065 model:V2065A device:PD2065 transport_id:4
        orga4pmzee4ts47t       device usb:338886656X product:atom model:M2004J7AC device:atom transport_id:24
        192.168.31.84:5555     device product:PD2065 model:V2065A device:PD2065 transport_id:5
    """

    for eachLine in deviceLines:
        if not eachLine:
            continue

        if "devices attached" in eachLine:
            continue

        foundDevice = re.search("(?P<devSerial>[\w\.\:]+)\s+device\s*(?P<devDetail>[\w\: ]+)?", eachLine)
        logging.debug("foundDevice=%s", foundDevice)
        # foundDevice=<re.Match object; span=(0, 101), match='2e2a0cb1               device usb:338952192X prod>
        if foundDevice:
            devSerial = foundDevice.group("devSerial")
            logging.debug("devSerial=%s", devSerial)
            # devSerial=2e2a0cb1
            if isGetDetail:
                devDetail = foundDevice.group("devDetail")
                logging.debug("devDetail=%s", devDetail)
                # devDetail=usb:338952192X product:PD2065 model:V2065A device:PD2065 transport_id:4
                keyValueIter = re.finditer("(?P<key>\w+):(?P<value>\w+)", devDetail) # <callable_iterator object at 0x10baa3a60>
                keyValueMatchList = list(keyValueIter)
                logging.debug("keyValueMatchList=%s", keyValueMatchList)
                # keyValueMatchList=[<re.Match object; span=(0, 14), match='usb:338952192X'>, <re.Match object; span=(15, 29), match='product:PD2065'>, <re.Match object; span=(30, 42), match='model:V2065A'>, <re.Match object; span=(43, 56), match='device:PD2065'>, <re.Match object; span=(57, 71), match='transport_id:4'>]
                detailInfoDict = {}
                for eachMatch in keyValueMatchList:
                    eachKey = eachMatch.group("key")
                    eachValue = eachMatch.group("value")
                    detailInfoDict[eachKey] = eachValue
                logging.debug("detailInfoDict=%s", detailInfoDict)
                # detailInfoDict={'usb': '338952192X', 'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '4'}
                curDevDetailDict = {
                    devSerial: detailInfoDict
                }
                logging.debug("curDevDetailDict=%s", curDevDetailDict)
                # curDevDetailDict={'2e2a0cb1': {'usb': '338952192X', 'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '4'}}
                deviceList.append(curDevDetailDict)
            else:
                deviceList.append(devSerial)

    logging.debug("deviceList=%s", deviceList)
    # deviceList=[{'2e2a0cb1': {'usb': '338952192X', 'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '4'}}, {'orga4pmzee4ts47t': {'usb': '338886656X', 'product': 'atom', 'model': 'M2004J7AC', 'device': 'atom', 'transport_id': '24'}}, {'192.168.31.84:5555': {'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '5'}}]
    # ['2e2a0cb1', 'orga4pmzee4ts47t', '192.168.31.84:5555']
    return deviceList
```

调用：

```python
deviceDetailList = self.getAndroidDeviceList(isGetDetail=False)
# ['2e2a0cb1', 'orga4pmzee4ts47t', '192.168.31.84:5555']
```

或：

```python
deviceDetailList = self.getAndroidDeviceList(isGetDetail=True)
# [{'2e2a0cb1': {'usb': '338952192X', 'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '4'}}, {'orga4pmzee4ts47t': {'usb': '338886656X', 'product': 'atom', 'model': 'M2004J7AC', 'device': 'atom', 'transport_id': '24'}}, {'192.168.31.84:5555': {'product': 'PD2065', 'model': 'V2065A', 'device': 'PD2065', 'transport_id': '5'}}]
```

## 检测安卓设备是否连接

```python
def isAndroidUsbConnected(self, deviceSerialId):
    """Check whether android device is currently USB wired connected or not

    Args:
        deviceSerialId (str): android devivce serial id
    Returns:
        connected or not (bool)
    Raises:
    Examples:
        input: "orga4pmzee4ts47t"
        output: True
    """
    isUsbConnected = False
    isRealSerialId = re.search("\w+", deviceSerialId)
    if not isRealSerialId:
        # makesure is not wifi, such as: 192.168.31.84:5555
        logging.error("Invalid android USB wired connected device serial id %s", deviceSerialId)
        return isUsbConnected

    deviceDetailList = self.getAndroidDeviceList(isGetDetail=True)
    for eachDevDetailDict in deviceDetailList:
        curDevSerialStr, curDevDetailDict = list(eachDevDetailDict.items())[0]
        if deviceSerialId == curDevSerialStr:
            detailInfoKeyList = list(curDevDetailDict.keys())
            # ['usb', 'product', 'model', 'device', 'transport_id']
            if "usb" in detailInfoKeyList:
                isUsbConnected = True
            break

    return isUsbConnected
```

调用：

```python
deviceId = "orga4pmzee4ts47t"
isUsbConnected = self.isAndroidUsbConnected(deviceId)
```

## 用adb通过WiFi连接设备

```python
def androidConnectWiFiDevice(self, wifiSerial):
    """Use Android `adb connect` to connect WiFi wireless devive

    Args:
        wifiSerial (str): android devivce WiFi serial, eg: 192.168.31.84:5555
    Returns:
        connect ok or not (bool)
    Raises:
    Examples:
        input: "192.168.31.84:5555"
        output: True
    """
    isConnectOk = False

    adbConnectCmd = "adb connect %s" % wifiSerial
    logging.info("Try connect Android device: %s", adbConnectCmd)
    # os.system(adbConnectCmd) # when failed, will wait too long time: ~ 1 minutes
    cmdOutputStr = CommonUtils.get_cmd_lines(adbConnectCmd, text=True, timeout=1)
    logging.info("console output: %s", cmdOutputStr)
    # connected to 192.168.31.84:5555
    # already connected to 192.168.31.84:5555
    # failed to connect to '192.168.31.84:5555': Operation timed out
    # "failed to connect to '192.168.31.84:5555': Connection refused\n"
    # err=Command 'adb connect 192.168.31.84:5555' timed out after 1 seconds when run cmd=adb connect 192.168.31.84:5555
    if cmdOutputStr:
        if "connected" in cmdOutputStr:
            isConnectOk = True
        elif ("failed" in cmdOutputStr) or ("timed out" in cmdOutputStr):
            isConnectOk = False
    else:
        isConnectOk = False

    return isConnectOk
```

调用：

```python
devWifiSerialId = "192.168.31.84:5555"
isWiFiConnected = self.androidConnectWiFiDevice(devWifiSerialId)
```

## 获取当前正在运行的app和页面activity

```python
def get_PackageActivity_Android(self):
    # adb直接获取当前活跃app及activity
    package, activity = "",""
    cmds = ['dumpsys activity |grep {}'.format(item) for item in ['mFocusedActivity','mResumedActivity']]
    for cmd in cmds:
        output = self.driver.shell(cmd).output
        result = re.search("u0(.*?)/", output)
        package = result.group(1).strip() if result else ""
        result = re.search("/(.*?)\s", output)
        activity = result.group(1).strip() if result else ""
        if package and activity:
            return package, activity
    return package, activity
```

调用：

```python
package, activity = self.get_PackageActivity()
```

## 获取已安装app列表

```python
def get_packages(self):
    # 获取已安装的app的appPackage列表
    if isinstance(self.driver, u2.UIAutomatorServer):
        text = self.driver.shell("pm list packages")[0]
        return re.findall(':(.*?)\n',text)
    else:
        cmd = 'adb -s {0} shell pm list packages'.format(self.device)
        lines = CommonUtils.get_cmd_lines(cmd)
        return {line.split(":")[1].strip() for line in lines}
```

调用：

```python
packages = self.get_packages()
```

## 安装安卓app

```python
def install_app_Android(self, item, packages=None):
    if packages is None:
        packages = self.get_packages()
    if item[1] in packages:
        logging.info("AppName {0} is alread installed".format(item[0]))
    else:
        logging.info("start to install app in {}".format(os.path.basename(self.arg_options.task)))
        os.system("adb -s {0} install {1}".format(self.device, item[3]))
```

调用：

```python
def install_app(self, item, packages=None):
    # 安装app
    if self.isAndroid:
        return self.install_app_Android(item, packages)
```

## 卸载安卓app

```python
def uninstall_app(self,item):
    # 卸载安装包
    os.system("adb -s {0} uninstall {1}".format(self.device, item[1]))
    logging.info("uninstall app {} end".format(item[1]))
```

调用：

```python
    if item[1] in packages:
        self.uninstall_app(item)
```

## 判断屏幕是否已解锁

```python
def is_device_unlock_Android(self,device):
    os.system('adb -s {} shell input keyevent 3'.format(device))
    time.sleep(1)
    # cmds = [
    # 	'adb -s {} shell dumpsys window policy | grep isStatusBarKeyguard'.format(device),
    # 	'adb -s {} shell dumpsys window policy | grep mShowingLockscreen'.format(device),
    # 	'adb -s {} shell dumpsys window policy | grep mDreamingLockscreen'.format(device),
    # ]
    # for cmd in cmds:
    # 	text = CommonUtils.get_cmd_lines(cmd,text=True)
    # 	if text and "=true" in text:
    # 		logging.info("start to unlock device {}".format(device))
    # 		return False

    # cmds = [
    # 	'adb -s {} shell dumpsys power | grep mHoldingDisplaySuspendBlocker'.format(device),
    # ]
    # for cmd in cmds:
    # 	text = CommonUtils.get_cmd_lines(cmd,text=True)
    # 	if text and "=false" in text:
    # 		# '  mHoldingDisplaySuspendBlocker=false\n'
    # 		logging.info("start to unlock device {}".format(device))
    # 		return False

    checkCmds = 'adb -s {} shell dumpsys window | grep mDreamingLockscreen'.format(device)
    text = CommonUtils.get_cmd_lines(checkCmds, text=True)
    if text and "mDreamingLockscreen=true" in text:
        # '    mShowingDream=false mDreamingLockscreen=true mDreamingSleepToken=null\n'
        logging.info("start to unlock device {}".format(device))
        return False

    logging.info("device {} is already unlock".format(device))
    return True
```

调用：

```python
if self.isAndroid:
    return self.is_device_unlock_Android(device)
```

## 获取安卓手机电量

```python
def get_device_electricity_Android(self):
    shell_cmd = 'dumpsys battery | grep level'
    adb_cmd = 'adb -s {0} shell {1}'.format(self.device,shell_cmd)
    # level = self.driver.shell(shell_cmd).output if isinstance(self.driver, u2.UIAutomatorServer) else CommonUtils.get_cmd_lines(adb_cmd,text=True)
    level = self.driver.shell(shell_cmd).output if isinstance(self.driver, u2.Device) else CommonUtils.get_cmd_lines(adb_cmd,text=True)
    result = re.search("\d+",level)
    ratio = int(result.group()) if result else 100
    return ratio
```

调用：

```python
batteryElectricityPercentInt = self.get_device_electricity_Android()
```

## 获取安卓手机名

```python
def get_phone_name_Android(self):
    # cmd = 'adb -s {} shell getprop ro.product.model'.format(self.device)
    cmd = 'adb -s {} shell getprop ro.product.name'.format(self.device)
    text = CommonUtils.get_cmd_lines(cmd,text=True)
    # https://miuiver.com/xiaomi-device-codename/
    # 	begonia -> 红米Note 8内部代号为“begonia”
    return re.sub("\s","",text)
    # isRunCmdOk, outputText = self.getCommandOutput(cmd)
    # if isRunCmdOk:
    # 	phoneName = outputText
    # else:
    # 	phoneName = ""
    # return phoneName
```

调用：

```python
if self.isAndroid:
    return self.get_phone_name_Android()
```

