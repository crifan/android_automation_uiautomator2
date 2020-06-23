# adb

此处整理uiautomator2开发期间，用到的和`adb`相关的东西。

## 解锁屏幕

安卓手机，华为的DIG-AL00，想要从锁屏界面，进入（等待输入密码的）解锁界面，可以用：

```bash
adb -s DWH9X17124W03779 shell input swipe 300 300 500 1000 100
```

或：

```bash
adb -s DWH9X17124W03779 shell input touchscreen swipe 300 300 500 1000 100
```

其中：

* `DWH9X17124W03779`：是手机的序列号
* `300 300 500 1000 100`：
    * `300 300 500 1000`：是屏幕坐标：X，Y，宽，高
    * `100`：滑动时间，单位毫秒

即可实现进入解锁界面。

输入文字（密码）：

```bash
adb -s DWH9X17124W03779 shell input text 1234
```

用于解锁手机。

特殊：

* 华为畅享6S DIG-AL00：不支持
    * 相关信息
        * android版本:6.0
        * 系统：EMUI 4.1
    * 原因：估计是系统问题
    * 解决办法：无法解决

详见：

【无法解决】adb发送密码无法解锁安卓手机屏幕


## adb shell中的am start命令

android的adb调试工具，有个shell，可以执行很多命令。

其内部都是调用对应的子工具去处理具体功能的。

此处相关的有：

* 调用 Activity 管理器 (`am`)
    * Activity Manager
* 调用软件包管理器 (`pm`)
* 调用设备政策管理器 (`dpm`)

其中am的解释是：

在 adb shell 中，您可以使用 Activity 管理器 (am) 工具发出命令以执行各种系统操作，如启动某项 Activity、强行停止某个进程、广播 intent、修改设备屏幕属性，等等。

在 shell 中，相应的语法为：

```bash
am command
```

您也可以直接从 adb 发出 Activity 管理器命令，无需进入远程 shell。例如：

```bash
adb shell am start -a android.intent.action.VIEW
```

具体参数含义解释：

command的语法=可用的 Activity 管理器命令

有很多，其中的start的语法是：

```bash
start [options] intent
```

* options=选项，包括：
    * -D：启用调试。
    * -W：等待启动完成。
    * --start-profiler file：启动分析器并将结果发送到 file。
    * -P file：类似于 --start-profiler，但当应用进入空闲状态时分析停止。
    * -R count：重复启动 Activity count 次。在每次重复前，将完成顶层 Activity。
    * -S：启动 Activity 前强行停止目标应用。
    * --opengl-trace：启用对 OpenGL 函数的跟踪。
    * --user user_id | current：指定要作为哪个用户运行；如果未指定，则作为当前用户运行。
* intent：启动 intent 指定的 Activity。
    * （主要）语法是：
        * -a action
            * 指定 intent 操作，例如 android.intent.action.VIEW（只能声明一次）。
        * -d data_uri
            * 指定 intent 数据 URI，例如 content://contacts/people/1（只能声明一次）。
        * -t mime_type
            * 指定 intent MIME 类型，例如 image/png（只能声明一次）。
        * -c category
            * 指定 intent 类别，例如 android.intent.category.APP_CONTACTS。
        * -n component
            * 指定带有软件包名称前缀的组件名称以创建显式 intent，例如 com.example.app/.ExampleActivity。
        * -f flags
            * 将标记添加到 setFlags() 支持的 intent。
        * --esn extra_key
            * 添加一个空 extra。URI intent 不支持此选项。
        * -e | --es extra_key extra_string_value
            * 将字符串数据作为键值对添加进来。
        * --ez extra_key extra_boolean_value
            * 将布尔型数据作为键值对添加进来。
        * --ei extra_key extra_int_value
            * 将整型数据作为键值对添加进来。
        * --el extra_key extra_long_value
            * 将长整型数据作为键值对添加进来。
        * --ef extra_key extra_float_value
            * 将浮点型数据作为键值对添加进来。
        * --eu extra_key extra_uri_value
            * 将 URI 数据作为键值对添加进来。
        * --ecn extra_key extra_component_name_value
            * 添加组件名称，该名称作为 ComponentName 对象进行转换和传递。
        * --eia extra_key extra_int_value\[,extra_int_value...\]
            * 添加整数数组。
        * --ela extra_key extra_long_value\[,extra_long_value...\]
            * 添加长整数数组。
        * --efa extra_key extra_float_value\[,extra_float_value...\]
            * 添加浮点数数组。


此处的：
* `am start -a android.intent.action.MAIN -c android.intent.category.LAUNCHER -n com.tencent.mm/.ui.LauncherUI`
    * `am start`：启动
    * `-a android.intent.action.MAIN`：intent的动作是`android.intent.action.MAIN`
    * `-c android.intent.category.LAUNCHER`：intent类别是android.intent.category.LAUNCHER
    * `-n com.tencent.mm/.ui.LauncherUI`
        * 要启动的app=包名：`com.tencent.mm`
            * 也就是微信
        * 要启动的activity=界面=页面：`.ui.LauncherUI`
            * 也就是微信的主页面