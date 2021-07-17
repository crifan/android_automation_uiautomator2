# 等待元素出现

当想要实现，（点击搜索按钮等操作后）希望某个元素出现，表示页面加载成功，可以用：`exists`

举例：

```python
Browser_XiaomiBuiltin = "com.android.browser"
browserPackage = Browser_XiaomiBuiltin
# wait util browser launch complete -> appear 主页 tab
d(text="主页", packageName=browserPackage).exists(timeout=10)
```

可以实现：
等待足够长时间（最长10秒），直到出现 
* package是"com.android.browser"
  * 小米的内置浏览器
* text是"主页"

表示浏览器的左下角第一个tab

以此表示 浏览器启动完毕了

从而实现了，等待（浏览器启动和加载）完毕（直到出现某个页面上的元素）的效果

后记：后来发现，左下角第一个tab，有时候会变，比如变成了：`资讯`

所以去用最后下角的 我的 tab，这个不会变

代码改为：

```python
# wait util browser launch complete -> appear 我的 tab
# MustShowTabName = "主页"
MustShowTabName = "我的"
# d(text=MustShowTabName).exists(timeout=10)
d(text=MustShowTabName, packageName=browserPackage).exists(timeout=10)
```

即可。