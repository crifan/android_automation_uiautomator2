# android-uiautomator-server

https://github.com/openatx/android-uiautomator-server

其发布出的

https://github.com/openatx/android-uiautomator-server/releases

是2个apk：

* `app-uiautomator-test.apk`
* `app-uiautomator.apk`

其具体编译过程是：

```bash
$ ./gradlew build
$ ./gradlew packageDebugAndroidTest
```

会生成apk，而最终的2个apk是mv生成的。

详见：`.travis.yml`中的：

```yml
script:
    - "./gradlew build"
    - "./gradlew packageDebugAndroidTest"
before_deploy:
    - mv app/build/outputs/apk/debug/app-debug.apk app/build/outputs/apk/app-uiautomator.apk
    - mv app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk app/build/outputs/apk/app-uiautomator-test.apk
```
