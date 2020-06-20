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
    #     begonia -> 红米Note 8内部代号为“begonia”
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
