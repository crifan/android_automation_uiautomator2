# xml源码不是最新的

之前遇到过一个很特殊的问题（bug），用dump_hierarchy去获取页面xml源码，但是获取的xml不是当前手机中显示的最新的页面的源码，而是旧的。

比如：之前某次情况是，当前是百度首页，但是输入了内容，触发了搜索后，页面内容已变化

但是用代码：

```python
d.dump_hierarchy(compressed=False, pretty=False)
```

输出的安卓的页面的xml源码却不是最新的

原因：是之前遇到的，uiautomator2的bug

解决办法：重启uiautomator2的服务

代码：

```python
d.service("uiautomator").stop()
d.service("uiautomator").start()
time.sleep(1)
```

即可规避问题，然后获取到xml源码，就是最新的了，包含了百度搜索的结果了。

