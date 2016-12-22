# OkHttp使用手册

> [官方网站](http://square.github.io/okhttp/) | [GitHub](https://github.com/square/okhttp) | [Wiki](https://github.com/square/okhttp/wiki) | [Javadoc](http://square.github.io/okhttp/3.x/okhttp/)

*   索引
    *   [简介及快速上手](#1)
    *   [Call模型](#2)
    *   [基本使用方法](#3)
    *   [更多内容](#4)
    *   [FAQ](#10)

## 简介及快速上手

### 简介

OkHttp是[Square](https://github.com/square)出的Http通讯库，支持HTTP和HTTP/2，用于Android应用和Java应用。

OkHttp是非常优秀的Http通讯库，将Http连接中各种繁杂的问题，对并发的支持，对常见异常的处理等封装在底层，提供简单易用的API供应用中调用。与之相比，`HttpUrlConnection`的使用过于复杂，Appache的`HttpClient`在Android平台上的运行又有各种问题，在Android 6.0之后，已经将`HttpClient`库从SDK中删除，全面转向使用OkHttp。

目前有很多知名的Android三方框架都使用`OkHttp`作为网络连接的默认基栈，例如Volley，Glide，Retrofit等，从中也能看出学习OkHttp的必要性。

### 快速上手

##### 在项目中添加对OkHttp的依赖

在需要使用OkHttp的模块的build.gradle文件中，添加如下依赖：(版本号可能变更)

```
compile 'com.squareup.okhttp3:okhttp:3.2.0'

```

##### 初始化

OkHttp框架的核心类是OkHttpClient，此类可直接实例化。由于OkHttpClient内部处理了并发，多线程和Socket重用等问题，为了节省资源，整个应用中使用一个OkHttpClient对象即可，可以对它做Singleton封装。

```
OkHttpClient okHttpClient = new OkHttpClient();

```

##### Http请求的构建

代表Http请求的类是Request，该类使用构造器模式，最简单的构造GET请求如下：

```
Request request = new Request.Builder()
      .url(url)
      .build();

```

要构造Post请求，在构建Request时增加请求体即可：

```
RequestBody formBody = new FormEncodingBuilder()
    .add("name", "Cuber")
    .add("age", "26")
    .build();

Request request = new Request.Builder()
      .url(url)
      .post(RequestBody)
      .build();

```

##### Http请求的发送

请求的发送有两种形式，一种是直接同步执行，阻塞调用线程，直接返回结果；另一种是通过队列异步执行，不阻塞调用线程，通过回调方法返回结果。如下所示：

同步执行：

```
// 如果返回null，代表超时或没有网络连接
Response response = client.newCall(request).execute();

```

异步回调：

```
Response response = client.newCall(request).enqueue(new Callback() {

    @Override
    public void onFailure(Request request, IOException e) {
        //超时或没有网络连接
        //注意：这里是后台线程！
    }

    @Override
        public void onResponse(Response response) throws IOException {
        //成功
        //注意：这里是后台线程！
    }
});

```

以上这些就是快速上手OkHttp需要知道的全部内容，可以从中看出，它的API非常简单易用。

## Call模型

Http客户端的任务是处理请求和响应，这说起来简单，但实际过程很复杂。

*   请求：Http请求包含一个URL，请求方法(例如GET或者POST)，请求头。还可能包含请求体，可以是数据流也可以是指定的内容类别。
*   响应：用一个响应码来回应请求(例如200代表成功，404代表页面未找到)，响应头和响应体。

### 请求的重写

为了保证正确性和传输效率，OkHttp会在发送你的请求之前重写它，例如：

*   OkHttp可能会添加原始请求中缺失的头信息，包括`Content-Length`, `Transfer-Encoding`, `User-Agent`, `Host`, `Connection`, 和 `Content-Type`。
*   为了实现透明的响应压缩(transparent response compression)，OkHttp会增加`Accept-Encoding`头信息。
*   如果你收到了cookie，OkHttp会增加`Cookie`头信息。
*   某些请求可能会对响应做缓存。如果被缓存的响应不是最新的，OkHttp能做一个有条件的GET请求来下载更新后的响应。此功能需要添加`If-Modified-Since`和`If-None-Match`等头信息。

### 响应的重写

如果使用了透明压缩，OkHttp会去掉对应响应的`Content-Encoding`和`Content-Length`头信息，因为它们不能应用于解压后的响应体。

如果有条件的GET成功了，网络侧的响应和缓存的响应会被自动合并。

### 重定向

如果你请求的URL被移动了，服务器会返回类似于302这样的响应码，来指明新的URL。OkHttp能跟随新的URL，获取到最终的响应。

### 请求的重试

有时会发生连接失败：可能网络连接状况不好，或者服务器不可达。OkHttp会自动使用不同的路由来重试请求。

### Call模型

由于以上的重写，重定向和重试等操作，你的一个简单请求可能会产生多个请求和响应。OkHttp使用**Call**这一概念对此来建模：不论为了满足你的请求任务，中间做了多少次请求和响应，都算作一个**Call**。

**Call**有两种方式来执行：

*   同步方式：你的线程会被阻塞，知道响应可读。
*   异步方式：你在任意线程将请求排队，当响应可读时，会在另一个线程拿到回调。

Call可以在任意线程取消，如果请求没有完成，调用取消方法会导致请求失败，读写请求体和响应体的代码会产生IOException。

### Call的分派

对于同步调用，你自己需要负责对线程和并发请求的管理。太多的同时存在的连接会浪费资源，太少则会影响延迟性能。

对于异步调用，`Dispatcher`实现了对最大并发请求数的管理。你可以设置最大的单服务器并发数(默认是5)，和最大的总并发数(默认是64)。

## 基本使用方法

以下是一些简单的代码样例，用来阐明如何用OkHttp解决常见问题。

### 同步Get请求

```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://publicobject.com/helloworld.txt")
        .build();

    // 下载文件
    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    // 打印响应的头信息
    Headers responseHeaders = response.headers();
    for (int i = 0; i < responseHeaders.size(); i++) {
      System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
    }

    // 将文件内容作为字符串输出
    System.out.println(response.body().string());
}

```

响应体的`string()`方法对于小文件来说非常方便和高效，但如果响应体较大(例如大于1M)，要避免使用这一方法，因为它会将文件内容全部加载在内存上。在这种情况下，使用流来处理响应体。

### 异步Get请求

下载一个文件，当响应可读时获得回调。具体收到回调的时间是当响应头准备好时，读取响应体仍然可能会阻塞。当前OkHttp没有提供异步API来接受响应体的各部分。

```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    Request request = new Request.Builder()
            .url("http://publicobject.com/helloworld.txt")
            .build();

    client.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Request request, IOException throwable) {
            throwable.printStackTrace();
        }

        @Override
        public void onResponse(Response response) throws IOException {
            if (!response.isSuccessful())
                throw new IOException("Unexpected code " + response);

            Headers responseHeaders = response.headers();
            for (int i = 0; i < responseHeaders.size(); i++) {
                System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
            }

            System.out.println(response.body().string());
        }
    });
}

```

### 获取头信息

```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    Request request = new Request.Builder()
            .url("https://api.github.com/repos/square/okhttp/issues")
            .header("User-Agent", "OkHttp Headers.java")
            .addHeader("Accept", "application/json; q=0.5")
            .addHeader("Accept", "application/vnd.github.v3+json")
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println("Server: " + response.header("Server"));
    System.out.println("Date: " + response.header("Date"));
    System.out.println("Vary: " + response.headers("Vary"));
}

```

### Post字符串

以下的例子显示了如何Post一个字符串给服务器，同样，不要使用此API传送大文件(> 1M)。

```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    String postBody = ""
            + "Releases\n"
            + "--------\n"
            + "\n"
            + " * _1.0_ May 6, 2013\n"
            + " * _1.1_ June 15, 2013\n"
            + " * _1.2_ August 11, 2013\n";

    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
}

```

### Post流

下面的例子使用了Okio的buffered sink。如果想使用OutputStream，可以通过`BufferedSink.outputStream()`方法。

```
public static final MediaType MEDIA_TYPE_MARKDOWN
        = MediaType.parse("text/x-markdown; charset=utf-8");

public static final MediaType MEDIA_TYPE_MARKDOWN
        = MediaType.parse("text/x-markdown; charset=utf-8");

private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    RequestBody requestBody = new RequestBody() {
        @Override public MediaType contentType() {
            return MEDIA_TYPE_MARKDOWN;
        }

        @Override public void writeTo(BufferedSink sink) throws IOException {
            sink.writeUtf8("Numbers\n");
            sink.writeUtf8("-------\n");
            for (int i = 2; i <= 997; i++) {
                sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
            }
        }

        private String factor(int n) {
            for (int i = 2; i < n; i++) {
                int x = n / i;
                if (x * i == n) return factor(x) + " × " + i;
            }
            return Integer.toString(n);
        }
    };

    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(requestBody)
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
}

```

### Post文件

```
public static final MediaType MEDIA_TYPE_MARKDOWN
        = MediaType.parse("text/x-markdown; charset=utf-8");

private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    File file = new File("README.md");

    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
}

```

### Post form参数

```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    RequestBody formBody = new FormBody.Builder()
            .add("search", "Jurassic Park")
            .build();
    Request request = new Request.Builder()
            .url("https://en.wikipedia.org/w/index.php")
            .post(formBody)
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
}

```

### Post多部分请求

`MultipartBody.Builder`可以构建复杂的请求体，多部分请求体的每一个部分都是一个单一的请求体，可以定义它自身的请求头。

```
private static final String IMGUR_CLIENT_ID = "...";
private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");

private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
    RequestBody requestBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("title", "Square Logo")
            .addFormDataPart("image", "logo-square.png",
                    RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
            .build();

    Request request = new Request.Builder()
            .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
            .url("https://api.imgur.com/3/image")
            .post(requestBody)
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
}

```

### 使用Gson来解析JSON响应

以下代码使用Gson库来解析服务器响应的JSON信息，注意`ResponseBody.charStream()`方法使用响应头中的`Content-Type`字段来选择使用何种字符集，如果没有指明，默认将使用UTF-8。

```
private final OkHttpClient client = new OkHttpClient();
private final Gson gson = new Gson();

public void run() throws Exception {
    Request request = new Request.Builder()
            .url("https://api.github.com/gists/c2a7c39532239ff261be")
            .build();
    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    Gist gist = gson.fromJson(response.body().charStream(), Gist.class);
    for (Map.Entry<String, GistFile> entry : gist.files.entrySet()) {
        System.out.println(entry.getKey());
        System.out.println(entry.getValue().content);
    }
}

static class Gist {
    Map<String, GistFile> files;
}

static class GistFile {
    String content;
}

```

### Call的取消

使用`Call.cancel()`来停止一个执行中的请求。如果线程正在写请求或读响应，则会收到IOException。使用此方法，在一个Call已经不需要时取消它，可以节省网络流量。

## 更多内容

以上是对OkHttp的基本了解，更多内容包括：

*   处理鉴权和Cookie。
*   使用HTTPS。
*   使用拦截器。
*   等等...

## FAQ

#### OkHttp如何设置超时？

设置超时的方法很简单：

```
OkHttpClient client = new OkHttpClient.Builder()
    .connectTimeout(10, TimeUnit.SECONDS)  // 连接超时
    .writeTimeout(10, TimeUnit.SECONDS)    // Socket写超时
    .readTimeout(30, TimeUnit.SECONDS)     // Socket读超时
    .build();

```

注意`readTimeout`和`writeTimeout`被用于OkHttp内部的`Connection`类，用在`setSoTimeout`方法上来设置`Socket`。

另外，在2.5.0版本之后，读、写、连接超时的默认值是10s。

#### OkHttp是否支持自签名SSL证书？

支持，需要自定义`javax.net.ssl.SSLSocketFactory`实例，传给`OkHttpClient.setSslSocketFactory(SSLSocketFactory sslSocketFactory)`方法。可以使用来自你自己Keystore的证书。

#### OkHttp中，Callback的回调方法onFailure和onResponse是在主线程执行码？

不是，OkHttp是一个Java库，不是Android库，它对Android主线程一无所知，所以这两个回调方法自然是在后台线程执行。实际开发中，常常对OkHttp做二次封装，来直接将结果传递给主线程。

#### 在OkHttp中可以禁止自动重定向吗？

在2.3.0版本中提供了设置的方法：

```
final OkHttpClient client = new OkHttpClient();
client.setFollowRedirects(false);

```

但在3.x版本中这一方法不存在了，所以暂时没有简单的方法来直接禁止自动重定向。

#### OKHttp既有同步api也有异步api，考虑下面两种方法来实现异步请求，有什么区别？哪一种更好？

1.  使用AsyncTask和OKhttp同步api；
2.  直接使用OKHttp异步api；

这两种方法区别很大！

为了HTTP请求而使用AsyncTask，在Android中是很糟糕的做法。它会造成很多缺陷，最好避免这样做。例如，你无法在执行期间撤销一个请求。另外，AsyncTask的使用模式常常会泄露Activity的引用，这是Android开发中容易引起内存泄漏的罪魁祸首之一。

OKHttp的异步方法从多个方面来讲都优越得多：

1.  异步api支持本地撤销请求。如果请求是在连接过程中，Callback的引用会被释放，不会再被调用；如果请求还没有开始，则不会被执行。如果你使用的是HTTP/2或者SPDY协议，我们就能实质上的在请求过程中撤销它，节省流量和电量。
2.  异步api支持给多个请求添加标记，并在一个方法中撤销全部被标记的请求。例如，在一个Activity中做出的所有请求，都可以用该Activity的实例做标记。然后，在onPause或onStop方法中，你可以撤销所有用该Activity实例标记的请求。
3.  如果你在使用HTTP/2或者SPDY，多个请求或者响应是复用单一连接传输到服务器的，在这种情况下，使用异步调用机制比阻塞的方式要高效得多。所以，如果可以，尽量使用Call.enqueue！

更多问题见[stackoverflow](http://stackoverflow.com/questions/tagged/okhttp?sort=active)。