---
title: Android与Javascript交互
date: 2016-11-16 17:59:39
categories:
tags:
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.com/2016/11/16/2016-11-16-Android%E4%B8%8EJavascript%E4%BA%A4%E4%BA%92/) | [CSDN](http://blog.csdn.net/q4878802/article/details/53189882)

---


本篇参考[Android与HTML+JS交互入门](http://blog.csdn.net/leejizhou/article/details/50894531)

# 效果图

![效果图](http://img.blog.csdn.net/20161116184229121)

# 加载本地Html

``` java
contentWebView = (WebView) findViewById(R.id.webview);
// 加载Assets下的Html
contentWebView.loadUrl("file:///android_asset/html/test.html");
```

# 启用Javascript

``` java
contentWebView.getSettings().setJavaScriptEnabled(true);
contentWebView.addJavascriptInterface(this, "android");
```


# Android调用Javascript的方法

Javascript写法

``` html
<script type="text/javascript">
    function jsFunction(){
         document.getElementById("content").innerHTML = "JS方法被调用";
    }
    function jsFunctionArg(arg){
         document.getElementById("content").innerHTML = "JS方法被调用并收到参数：<br/>" + arg;
    }
</script>
```

Android写法

``` java
// 调用JS的jsFunction方法
contentWebView.loadUrl("javascript:jsFunction()");
// 调用JS的jsFunctionArg方法
contentWebView.loadUrl("javascript:jsFunctionArg('[Android传递过来的数据]')");
```

# Javascript调用Android的方法

Android方法

``` java
@JavascriptInterface
public void androidFunction() {
    Snackbar.make(contentWebView, "Android的方法被调用", Snackbar.LENGTH_SHORT).show();
}
```

``` java
@JavascriptInterface
public void androidFunction(String text) {
    Snackbar.make(contentWebView, "Android的方法被调用并收到参数 : \n" + text, Snackbar.LENGTH_SHORT).show();
}
```

Javascript调用

``` html
<input type="button" style="width:300px;height:50px;" value="JS调用Android方法" onclick="window.android.androidFunction()" />
```

``` html
<input type="button" style="width:300px;height:50px;" value="JS调用Android带有参数的方法" onclick="window.android.androidFunction('[JS传递过来的数据]')" />
```


----------------------------

# Code

## HTML

![XML](http://img.blog.csdn.net/20161116184259654)

``` html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=gb2312">
        <script type="text/javascript">
            function jsFunction(){
                 document.getElementById("content").innerHTML = "JS方法被调用";
            }
            function jsFunctionArg(arg){
                 document.getElementById("content").innerHTML = "JS方法被调用并收到参数：<br/>" + arg;
            }
        </script>
    </head>
    <body>
        <h1>HTML页面</h1>
        <hr/>
        <h3><div id="content">|</div></h3>
        <hr/>
        <input type="button" style="width:300px;height:50px;" value="JS调用Android方法" onclick="window.android.androidFunction()" />
        <br/>
        <input type="button" style="width:300px;height:50px;" value="JS调用Android带有参数的方法" onclick="window.android.androidFunction('[JS传递过来的数据]')" />
    </body>
</html>
```

## 测试类

``` java
package com.kongqw.kqwandroidjsdemo;

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.Menu;
import android.view.MenuItem;
import android.webkit.JavascriptInterface;
import android.webkit.WebView;

public class MainActivity extends AppCompatActivity {

    private WebView contentWebView;

    @SuppressLint({"JavascriptInterface", "SetJavaScriptEnabled", "AddJavascriptInterface"})
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        contentWebView = (WebView) findViewById(R.id.webview);
        // 加载Assets下的Html
        contentWebView.loadUrl("file:///android_asset/html/test.html");
        // 启用Javascript
        contentWebView.getSettings().setJavaScriptEnabled(true);
        contentWebView.addJavascriptInterface(this, "android");
    }


    @JavascriptInterface
    public void androidFunction() {
        Snackbar.make(contentWebView, "Android的方法被调用", Snackbar.LENGTH_SHORT).show();
    }

    @JavascriptInterface
    public void androidFunction(String text) {
        Snackbar.make(contentWebView, "Android的方法被调用并收到参数 : \n" + text, Snackbar.LENGTH_SHORT).show();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if (id == R.id.action_js_function1) {
            // 调用JS的jsFunction方法
            contentWebView.loadUrl("javascript:jsFunction()");
            return true;
        } else if (id == R.id.action_js_function2) {
            // 调用JS的jsFunctionArg方法
            contentWebView.loadUrl("javascript:jsFunctionArg('[Android传递过来的数据]')");
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```












