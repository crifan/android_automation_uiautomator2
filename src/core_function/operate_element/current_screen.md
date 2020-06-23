# 当前屏幕

针对于当前屏幕，最常见的几个动作是：

* 截图=截屏
* 获取(当前)页面源码(xml)

## 给当前屏幕截图

核心代码：

```python
fullImgFilePath = self.driver.screenshot(fullImgFilePath)
```

举例：

```python
fullImgFilePath = 'debug/GameScreenshot/20191209_171115.png'
fullImgFilePath = self.driver.screenshot(fullImgFilePath)
```

详见：

【已解决】uiautomator2中如何获取到当前画面的截图文件

## 获取当前屏幕画面对应的xml源码

对于下图中左边的登录界面：

![weditor_show_left_page_right_xml](../../assets/img/weditor_show_left_page_right_xml.png)

用：

```python
page_source = self.driver.dump_hierarchy(compressed=False, pretty=False)
```

导出的源码是：

```xml
<?xml version=1.0 encoding=UTF-8 standalone=yes ?>
<hierarchy rotation="1">
    <node index="0" text="" resource-id="" class="android.widget.FrameLayout" package="com.sy4399.pmxtyd2" content-desc="" checkable="false" checked="false" clickable="false" enabled="true" focusable="false" focused="false" scrollable="false" long-clickable="false" password="false" selected="false" bounds="[0,0][1196,720]">
...
                                                    <node NAF="true" index="4" text="" resource-id="" class="android.widget.ToggleButton" package="com.sy4399.pmxtyd2" content-desc="" checkable="true" checked="false" clickable="true" enabled="true" focusable="true" focused="false" scrollable="false" long-clickable="false" password="false" selected="false" bounds="[834,314][906,386]" />
                                                </node>
                                            </node>
                                            <node index="1" text="" resource-id="" class="android.widget.LinearLayout" package="com.sy4399.pmxtyd2" content-desc="" checkable="false" checked="false" clickable="false" enabled="true" focusable="false" focused="false" scrollable="false" long-clickable="false" password="false" selected="false" bounds="[268,440][928,530]">
                                                <node index="0" text="立即登录" resource-id="" class="android.widget.Button" package="com.sy4399.pmxtyd2" content-desc="" checkable="false" checked="false" clickable="true" enabled="true" focusable="true" focused="false" scrollable="false" long-clickable="true" password="false" selected="false" bounds="[268,440][928,530]" />
                                            </node>
...
                                            </node>
                                        </node>
                                    </node>
                                </node>
                            </node>
                            <node index="1" text="" resource-id="" class="android.view.View" package="com.sy4399.pmxtyd2" content-desc="" checkable="false" checked="false" clickable="false" enabled="true" focusable="false" focused="false" scrollable="false" long-clickable="false" password="false" selected="false" bounds="[218,80][978,86]" />
                        </node>
                    </node>
                </node>
            </node>
        </node>
    </node>
</hierarchy>
```
