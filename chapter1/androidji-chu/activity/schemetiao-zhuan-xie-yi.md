> 什么是URL Schema

URL Schema是一种页面内跳转机制，可以自定义Schema协议，很方便的实现App的定制化跳转；可以方便服务器指定跳转的页面，可以通过通知栏指定跳转页面，可以通过H5跳转页面等。

> URL Schema 应用场景

客户端向操作系统注册一个URL Schema，该地址用于从浏览器或其他App进行启动本应用。通过指定的 URL 字段，可以让应用在被调起后直接打开某些特定页面，比如商品详情页、活动详情页等等。也可以执行某些指定动作，如完成支付等。也可以在应用内通过 html 页来直接调用显示 app 内的某个页面。综上URL Schema使用场景大致分以下几种：

1. 服务器下发跳转路径，客户端根据服务器下发的跳转路径进行跳转
2. H5页面锚点，根据路径跳转到APP指定页面
3. APP端收到服务器PUSH通知栏消息，根据消息跳转指定页面
4. APP通过URL跳转到另一个APP指定路径

> 格式

xl://goods:8888/goodsDetail?goodsId=10011002

* xl:代表Schema协议名称
* goods代表该Schema作用于哪一个域
* goodsDetail:代表Schema指定的页面
* goodsId:代表传递的参数
* 8888:代表该路径的端口号 

> 如何使用

1.在AndroidManifest.xml中对&lt;activity /&gt;标签增加&lt;intent-filter /&gt;设置Schema

```
<activity
            android:name=".GoodsDetailActivity"

            android:theme="@style/AppTheme">

            <!--要想在别的App上能成功调起App，必须添加intent过滤器-->

            <intent-filter>

                <!--协议部分，随便设置-->

                <data
                    android:host="goods"
                    android:path="/goodsDetail"
                    android:port="8888"
                    android:scheme="xl" />

                <!--下面这几行也必须得设置-->

                <category android:name="android.intent.category.DEFAULT" />


                <action android:name="android.intent.action.VIEW" />


                <category android:name="android.intent.category.BROWSABLE" />


            </intent-filter>


</activity>
```

2.

