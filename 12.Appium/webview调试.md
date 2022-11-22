### webview

Android（6.0 以上）需要打开 webview 调试开关

```Java
if (Biuld.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
    WebView.setWebContentsDebuggingEnabled(true);
}
```

mumu模拟器无法获取到 webview 页面和窗口



参考文章：https://mp.weixin.qq.com/s?__biz=MzU2NzM4MTUxNw==&mid=2247483705&idx=1&sn=8be63f27f012c802ae809859c6219f19&chksm=fc9f5a5bcbe8d34d9ca81c2034719923ae3e7972ac1746e149c724333f190d53cb6bda23b8ea&token=1639160123&lang=zh_CN#rd