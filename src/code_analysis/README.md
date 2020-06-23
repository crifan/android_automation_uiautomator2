# 源码分析

折腾uiautomator2期间，分析了其中部分源码。现把过程整理如下供参考。

## uiautomator-server中最终的底层实现dumpWindowHierarchy的处理返回页面数据的逻辑

### 先启动了底层的jsonrpc的服务，监听发送过来的，要执行的动作

文件：`app/src/androidTest/java/com/github/uiautomator/stub/Stub.java`

```java
import com.googlecode.jsonrpc4j.JsonRpcServer;

public class Stub {
...
    int PORT = 9008;
    AutomatorHttpServer server = new AutomatorHttpServer(PORT);


    @Before
    public void setUp() throws Exception {
        launchService();
        JsonRpcServer jrs = new JsonRpcServer(new ObjectMapper(), new AutomatorServiceImpl(), AutomatorService.class);
...
        server.route("/jsonrpc/0", jrs);
        server.start();
    }
```

其中对于launchService：

```java
    private void launchService() throws RemoteException {
        UiDevice device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
        Context context = InstrumentationRegistry.getContext();
        device.wakeUp();


        // Wait for launcher
        String launcherPackage = device.getLauncherPackageName();
        Boolean ready = device.wait(Until.hasObject(By.pkg(launcherPackage).depth(0)), LAUNCH_TIMEOUT);
        if (!ready) {
            Log.i(TAG, "Wait for launcher timeout");
            return;
        }


        Log.d("Launch service");
        startMonitorService(context);
    }


    private void startMonitorService(Context context) {
        Intent intent = new Intent("com.github.uiautomator.ACTION_START");
        intent.setPackage("com.github.uiautomator"); // fix error: Service Intent must be explicit
        context.startService(intent);
    }
```

去启动了`com.github.uiautomator`，应该就是在后台运行的uiautomator的服务了。

而前面的`JsonRpcServer`的`jrs`，则是：

* 负责监听`/jsonrpc/0`
    * 对应着之前`uiautomator2`中发送过来的请求
        * `Shell$ curl -X POST -d 'b'{"jsonrpc": "2.0", "id": "1f056baf5d6b2ea2cb7e546efb7cd64f", "method": "dumpWindowHierarchy", "params": [true, null]}'' http://127.0.0.1:64445/jsonrpc/0`
        * 中的`jsonrpc/0`
* 其具体实现的类是`AutomatorServiceImpl`的`AutomatorService`
    * 下面就来介绍`AutomatorServiceImpl`

文件：`app/src/androidTest/java/com/github/uiautomator/stub/AutomatorServiceImpl.java`

```java
public class AutomatorServiceImpl implements AutomatorService {

    /**
     * It's to test if the service is alive.
     *
     * @return 'pong'
     */
    @Override
    public String ping() {
        return "pong";
    }

    /**
     * Get the device info.
     *
     * @return device info.
     */
    @Override
    public DeviceInfo deviceInfo() {
        return DeviceInfo.getDeviceInfo();
    }

...
}
```

上面是最基本的几个函数：

* `ping`
    * 返回`pong`
        * 表示服务还在，有效、alive
* `deviceInfo`
    * 对应着之前调试：
        * `d = u2.connect('8c8a4d4d')`
    * 期间输出的：
        * `conn=<urllib3.connection.HTTPConnection object at 0x1077f4be0>,method=POST,url=/jsonrpc/0,timeout_obj=Timeout(connect=2, read=2, total=None),body={"jsonrpc": "2.0", "id": 1, "method": "deviceInfo"},headers={'User-Agent': 'python-requests/2.22.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive', 'Content-Length': '51'},chunked=False`
    * 中的
        * `"method": "deviceInfo"`
    * 用于返回设备信息

而AutomatorServiceImpl中海油其他很多很多功能的具体实现。下面分别介绍一下之前接触过的。

```java
public boolean click(int x, int y) {
public boolean drag(int startX, int startY, int endX, int endY, int steps) throws NotImplementedException {
public boolean swipe(int startX, int startY, int endX, int endY, int steps) {
...
```

都是常见的基础功能。

```java
    // Multi touch is a little complicated
    @Override
    public boolean injectInputEvent(int action, float x, float y, int metaState) {
        MotionEvent e = MotionEvent.obtain(SystemClock.uptimeMillis(),
                SystemClock.uptimeMillis(),
                action, x, y, metaState);
        e.setSource(InputDevice.SOURCE_TOUCHSCREEN);
        boolean b = uiAutomation.injectInputEvent(e, true);
        e.recycle();
        return b;
    }
```

之前就遇到过多次，上层调用一些函数会报错，其中就会提到这个

比如：

【已解决】python的uiautomator2报错：uiautomator2.exceptions.JsonRpcError -32601 Method not found data injectInputEvent

中的

```java
    obj.jsonrpc.injectInputEvent(ACTION_DOWN, x, y, 0)
```

其他还有很多很多：

```java
    /**
     * Simulates a short press using key name.
     *
     * @param key possible key name is home, back, left, right, up, down, center, menu, search, enter, delete(or del), recent(recent apps), volume_up, volume_down, volume_mute, camera, power
     * @return true if successful, else return false
     * @throws RemoteException
     */
    @Override
    public boolean pressKey(String key) throws RemoteException {
        boolean result;
        key = key.toLowerCase();
        if ("home".equals(key)) result = device.pressHome();
        else if ("back".equals(key)) result = device.pressBack();
        else if ("left".equals(key)) result = device.pressDPadLeft();
        else if ("right".equals(key)) result = device.pressDPadRight();
        else if ("up".equals(key)) result = device.pressDPadUp();
        else if ("down".equals(key)) result = device.pressDPadDown();
        else if ("center".equals(key)) result = device.pressDPadCenter();
        else if ("menu".equals(key)) result = device.pressMenu();
        else if ("search".equals(key)) result = device.pressSearch();
        else if ("enter".equals(key)) result = device.pressEnter();
        else if ("delete".equals(key) || "del".equals(key)) result = device.pressDelete();
        else if ("recent".equals(key)) result = device.pressRecentApps();
        else if ("volume_up".equals(key)) result = device.pressKeyCode(KeyEvent.KEYCODE_VOLUME_UP);
        else if ("volume_down".equals(key))
            result = device.pressKeyCode(KeyEvent.KEYCODE_VOLUME_DOWN);
        else if ("volume_mute".equals(key))
            result = device.pressKeyCode(KeyEvent.KEYCODE_VOLUME_MUTE);
        else if ("camera".equals(key)) result = device.pressKeyCode(KeyEvent.KEYCODE_CAMERA);
        else result = "power".equals(key) && device.pressKeyCode(KeyEvent.KEYCODE_POWER);


        return result;
    }


 
public boolean pressKeyCode(int keyCode) {
public boolean pressKeyCode(int keyCode, int metaState) {

public void clearTextField(Selector obj) throws UiObjectNotFoundException {




    /**
     * Reads the text property of the UI element
     *
     * @param obj the selector of the UiObject.
     * @return text value of the current node represented by this UiObject
     * @throws UiObjectNotFoundException
     */
    @Override
    public String getText(Selector obj) throws UiObjectNotFoundException {
        if (obj.toUiObject2() == null) {
            return device.findObject(obj.toUiSelector()).getText();
        } else {
            return obj.toUiObject2().getText();
        }
    }


    /**
     * Sets the text in an editable field, after clearing the field's content. The UiSelector selector of this object must reference a UI element that is editable. When you call this method, the method first simulates a click() on editable field to set focus. The method then clears the field's contents and injects your specified text into the field. If you want to capture the original contents of the field, call getText() first. You can then modify the text and use this method to update the field.
     *
     * @param obj  the selector of the UiObject.
     * @param text string to set
     * @return true if operation is successful
     * @throws UiObjectNotFoundException
     */
    @Override
    public boolean setText(Selector obj, String text) throws UiObjectNotFoundException {
        try {
            obj.toUiObject2().click();
            obj.toUiObject2().setText(text);
            return true;
        } catch (NullPointerException | StaleObjectException e) {
            return device.findObject(obj.toUiSelector()).setText(text);
        }
    }


    /**
     * Performs a click at the center of the visible bounds of the UI element represented by this UiObject.
     *
     * @param obj the target ui object.
     * @return true id successful else false
     * @throws UiObjectNotFoundException
     */
    @Override
    public boolean click(Selector obj) throws UiObjectNotFoundException {
        if (obj.toUiObject2() == null) {
            return device.findObject(obj.toUiSelector()).click();
        } else {
            obj.toUiObject2().click();
            return true;
        }
    }


    /**
     * Clicks the bottom and right corner or top and left corner of the UI element
     *
     * @param obj    the target ui object.
     * @param corner "br"/"bottomright" means BottomRight, "tl"/"topleft" means TopLeft, "center" means Center.
     * @return true on success
     * @throws UiObjectNotFoundException
     */
    @Override
    public boolean click(Selector obj, String corner) throws UiObjectNotFoundException {
        return click(device.findObject(obj.toUiSelector()), corner);
    }


    private boolean click(UiObject obj, String corner) throws UiObjectNotFoundException {
        if (corner == null) corner = "center";
        corner = corner.toLowerCase();
        if ("br".equals(corner) || "bottomright".equals(corner)) return obj.clickBottomRight();
        else if ("tl".equals(corner) || "topleft".equals(corner)) return obj.clickTopLeft();
        else if ("c".equals(corner) || "center".equals(corner)) return obj.click();
        return false;
    }


public boolean dragTo(Selector obj, Selector destObj, int steps) throws UiObjectNotFoundException, NotImplementedException {




    /**
     * Performs the swipe up/down/left/right action on the UiObject
     *
     * @param obj   the target ui object.
     * @param dir   "u"/"up", "d"/"down", "l"/"left", "r"/"right"
     * @param steps indicates the number of injected move steps into the system. Steps are injected about 5ms apart. So a 100 steps may take about 1/2 second to complete.
     * @return true of successful
     * @throws UiObjectNotFoundException
     */
    @Override
    public boolean swipe(Selector obj, String dir, int steps) throws UiObjectNotFoundException {
        return swipe(device.findObject(obj.toUiSelector()), dir, steps);
    }


    private boolean swipe(UiObject item, String dir, int steps) throws UiObjectNotFoundException {
        dir = dir.toLowerCase();
        boolean result = false;
        if ("u".equals(dir) || "up".equals(dir)) result = item.swipeUp(steps);
        else if ("d".equals(dir) || "down".equals(dir)) result = item.swipeDown(steps);
        else if ("l".equals(dir) || "left".equals(dir)) result = item.swipeLeft(steps);
        else if ("r".equals(dir) || "right".equals(dir)) result = item.swipeRight(steps);
        return result;
    }

...
...
...
```

其他更多函数就不贴代码了。

### 底层调用dumpWindowHierarchy，处理，返回数据

如上所述，AutomatorServiceImpl.java 中的很多功能函数，此处最关心的dumpWindowHierarchy了：

```java
    /**
     * Helper method used for debugging to dump the current window's layout hierarchy. The file root location is /data/local/tmp
     *
     * @param compressed use compressed layout hierarchy or not using setCompressedLayoutHeirarchy method. Ignore the parameter in case the API level lt 18.
     * @param filename   the filename to be stored. @deprecated
     * @return the absolute path name of dumped file.
     */
    @Deprecated
    @Override
    public String dumpWindowHierarchy(boolean compressed, String filename) {
        return dumpWindowHierarchy(compressed);
    }

    /**
     * Helper method used for debugging to dump the current window's layout hierarchy.
     *
     * @param compressed use compressed layout hierarchy or not using setCompressedLayoutHeirarchy method. Ignore the parameter in case the API level lt 18.
     * @return the absolute path name of dumped file.
     */
    @Override
    public String dumpWindowHierarchy(boolean compressed) {
        device.setCompressedLayoutHeirarchy(compressed);
        ByteArrayOutputStream os = null;
        try {
            os = new ByteArrayOutputStream();
            AccessibilityNodeInfoDumper.dumpWindowHierarchy(device, os);
//            device.dumpWindowHierarchy(os);


            return os.toString("UTF-8");
        } catch (IOException e) {
            Log.d("dump Window Hierarchy got IOException " + e);
        }finally {
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                    //ignore
                }
            }
        }


        return null;
    }
```

前一个：

```java
public String dumpWindowHierarchy(boolean compressed, String filename) {
```

已废弃。

后一个，核心是调用：

```java
AccessibilityNodeInfoDumper.dumpWindowHierarchy(device, os);
```

`app/src/androidTest/java/com/github/uiautomator/stub/AccessibilityNodeInfoDumper.java`

```java
    public static void dumpWindowHierarchy(UiDevice device, OutputStream out) throws IOException {
        XmlSerializer serializer = Xml.newSerializer();
        serializer.setFeature("http://xmlpull.org/v1/doc/features.html#indent-output", true);
        serializer.setOutput(out, "UTF-8");
        serializer.startDocument("UTF-8", true);
        serializer.startTag("", "hierarchy");
        serializer.attribute("", "rotation", Integer.toString(device.getDisplayRotation()));
        AccessibilityNodeInfo[] arr$ = getWindowRoots(device); // device.getWindowRoots();
        int len$ = arr$.length;


        for (int i$ = 0; i$ < len$; ++i$) {
            AccessibilityNodeInfo root = arr$[i$];
            dumpNodeRec(root, serializer, 0, device.getDisplayWidth(), device.getDisplayHeight());
        }


        serializer.endTag("", "hierarchy");
        serializer.endDocument();
    }
```

最终返回的内容，就是此处的dumpWindowHierarchy函数的处理，生成xml内容后，所返回的。

比如某次调试过程：

jsonrpc的调用：

```bash
[191120 10:17:07][__init__.py 493] jsonrpc_call: jsonrpc_url=http://127.0.0.1:64445/jsonrpc/0, method=dumpWindowHierarchy, params=(True, None), http_timeout=60
```

底层发送的请求是：

```bash
Shell$ curl -X POST -d 'b'{"jsonrpc": "2.0", "id": "5a175f3159cc1aa2e27f1cb68f5a0509", "method": "dumpWindowHierarchy", "params": [true, null]}'' http://127.0.0.1:64445/jsonrpc/0
```

最终返回的结果是：

```bash
Output> {"jsonrpc":"2.0","id":"5a175f3159cc1aa2e27f1cb68f5a0509","result":"<?xml version='1.0' encoding='UTF-8' standalone='yes' ?><hierarchy rotation=\"0\"><node index=\"0\" text=\"\" resource-id=\"\" class=\"android.widget.FrameLayout\" package=\"com.tencent.mm\" content-desc=\"当前所在页面,搜一搜\" checkable=\"false\" checked=\"false\" clickable=\"false\" enabled=\"true\" focusable=\"false\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[0,0][720,1208]\"><node index=\"0\" text=\"\" resource-id=\"com.tencent.mm:id/om\" class=\"android.widget.FrameLayout\" package=\"com.tencent.mm\" content-desc=\"containerFrameLayout\" checkable=\"false\" checked=\"false\" clickable=\"false\" enabled=\"true\" focusable=\"false\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[0,134][720,1208]\"><node index=\"0\" text=\"\" resource-id=\"com.tencent.mm:id/boe\" class=\"android.widget.FrameLayout\" package=\"com.tencent.mm\" content-desc=\"containerFrameLayout\" checkable=\"false\" checked=\"false\" clickable=\"false\" enabled=\"true\" focusable=\"false\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[0,134][720,1208]\"><node NAF=\"true\" index=\"0\" text=\"\" resource-id=\"\" class=\"com.tencent.tbs.core.webkit.WebView\" package=\"com.tencent.mm\" content-desc=\"\" checkable=\"false\" checked=\"false\" clickable=\"true\" enabled=\"true\" focusable=\"true\" focused=\"true\" scrollable=\"false\" long-clickable=\"true\" password=\"false\" selected=\"false\" bounds=\"[0,134][720,1208]\" /></node></node><node index=\"1\" text=\"\" resource-id=\"com.tencent.mm:id/m4\" class=\"android.widget.LinearLayout\" package=\"com.tencent.mm\" content-desc=\"\" checkable=\"false\" checked=\"false\" clickable=\"true\" enabled=\"true\" focusable=\"false\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[0,48][71,134]\"><node index=\"0\" text=\"\" resource-id=\"com.tencent.mm:id/m5\" class=\"android.widget.ImageView\" package=\"com.tencent.mm\" content-desc=\"返回\" checkable=\"false\" checked=\"false\" clickable=\"false\" enabled=\"true\" focusable=\"false\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[0,48][71,134]\" /></node><node index=\"3\" text=\"gh_cfcfcee032cc\" resource-id=\"com.tencent.mm:id/m7\" class=\"android.widget.EditText\" package=\"com.tencent.mm\" content-desc=\"\" checkable=\"false\" checked=\"false\" clickable=\"true\" enabled=\"true\" focusable=\"true\" focused=\"false\" scrollable=\"false\" long-clickable=\"true\" password=\"false\" selected=\"false\" bounds=\"[116,48][706,134]\" /><node NAF=\"true\" index=\"4\" text=\"\" resource-id=\"com.tencent.mm:id/m3\" class=\"android.widget.ImageButton\" package=\"com.tencent.mm\" content-desc=\"\" checkable=\"false\" checked=\"false\" clickable=\"true\" enabled=\"true\" focusable=\"true\" focused=\"false\" scrollable=\"false\" long-clickable=\"false\" password=\"false\" selected=\"false\" bounds=\"[663,69][706,112]\" /></node></hierarchy>"}
```

可见其中的xml头部的内容：

```xml
<?xml version='1.0' encoding='UTF-8' standalone='yes' ?><hierarchy rotation="0">
```

就是上面的`XmlSerializer`的代码所生成的。

而其他的node节点，则是dumpNodeRec所生成的。

由此，后续深入研究，才知道，最终返回的节点中，如果符合NAF条件，则会被忽略其下内容，最终返回一个NAF=“true”的节点，导致后续只返回部分页面内容的。

具体细节详见：

* 【未解决】uiautomator2中dump_hierarchy中只能获取到页面的部分的xml源码
* 【已解决】搞懂uiautomator-server中最终的底层实现dumpWindowHierarchy的处理返回页面数据的逻辑
