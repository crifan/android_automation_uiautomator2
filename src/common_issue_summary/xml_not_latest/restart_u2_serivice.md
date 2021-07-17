# 重启服务

当偶尔遇到uiautomator2本身出问题，而后台服务停止或者异常时，可以去重启uiautomator2的服务。

经研究，总结出相关代码：

```python
def u2ServiceRestart(self):
    """ restart uiautomator2 service """
    # self.driver.reset_uiautomator()
    # self.driver.service.stop()
    # self.driver.service.start()
    self.driver.service("uiautomator").stop()
    self.driver.service("uiautomator").start()
    # self.driver.uiautomator.stop()
    # self.driver.uiautomator.start()
    time.sleep(1)
```

调用举例：

```python
def wait_AccountSearched(self, account):
    ...
    if self.isAndroid:
        # Note1: 
        # for 华为畅享6S android 6: following can not get latest page xml after search weixin public account
        # here reset uiautomator2 is workaround for later d.dump_hierarchy get latest  page xml
        # Note2: 小米9 Android 10 / 红米 Note8 Pro Android 9: Not need reset uiautomator2 anymore
        isNeedRestartU2 = False

        curAndroidVersionFloat = self.getAndroidVersion()
        ANDROID_VERSION_NEED_RESTART_U2 = 7.0
        if curAndroidVersionFloat <= ANDROID_VERSION_NEED_RESTART_U2:
            isNeedRestartU2 = True

        if isNeedRestartU2:
            self.u2ServiceRestart()
```

