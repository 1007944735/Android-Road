## Android与H5互调

Android与H5互调实质上是java与js的互相调用

### 1.Android调用H5的方法

Android调用H5的方法比较简单，在H5上定义一个方法比如

```
<script type="text/javascript">
    function javaCallJs(arg){
         document.getElementById("content").innerHTML =
             ("欢迎："+arg );
    }
</script>
```

在Android中使用javascript：h5定义的方法名（字符类型的入参）

```
webView.loadUrl("javascript:javaCallJs('测试')");
```

### 2.H5调用Android

H5调用Android原生代码，需要调用WebView.addJavascriptInterface()

```
webView.addJavascriptInterface(new AndroidAndJsInterface(),"Android");

private class AndroidAndJsInterface {
        @JavascriptInterface
        public void showToast(){
            Toast.makeText(WebJsActivity.this, "原生代码", Toast.LENGTH_SHORT).show();
        }
    }
```

然后在h5页面的中使用window.对象名.调用方法名

```
<input type="button" value="点击Android被调用" onclick="window.Android.showToast()"/>
```