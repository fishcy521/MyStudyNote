# Retrofit使用文档

## Retrofit简介

### 依赖

> compile 'com.squareup.retrofit2:retrofit:2.0.2'

如果使用了混淆功能，在proguard文件中添加如下配置：

```
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }

```

### 功能

Retrofit将HTTP API转换成Java接口：

```
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}

```

核心类`Retrofit`能生成上面这个`GitHubService`接口的实现：

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);

```

`GitHubService`创建出的`Call`可以做同步或异步HTTP请求：

```
Call<List<Repo>> repos = service.listRepos("octocat");

```

### 主要特性

*   URL参数的替换和对查询参数的支持
*   对象到请求体的转换(例如JSON)
*   多部分请求体(Multipart request body)和文件上传

## API申明

通过对接口中方法和方法参数添加注解，来告诉Retrofit如何处理该请求。

### HTTP方法的注解

接口中的每一个方法，必须有一个注解来指明其对应的HTTP方法以及相对路径。Retrofit支持五种HTTP方法的注解：`GET`，`POST`，`PUT`，`DELETE`，`HEAD`。相对URL路径也在注解中指定：

```
@GET("users/list")

```

users/list

也可以将查询参数直接写死在URL中：

```
@GET("users/list?sort=desc")

```

users/list?sort=desc

### URL处理

HTTP请求的URL常常需要动态的设置，可以通过“可替换块”和对方法参数的注解来实现。“可替换块”是形如`"{...}"`的字符串，相对应的方法参数使用`@Path`注解。

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);

```

调用groupList(10) -----> group/**10**/users

查询参数也可以动态添加：

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

```

调用groupList(10,"desc") ---> group/**10**/users**?sort=desc**

对于复杂的查询参数，可以使用`Map`:

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);

```

### 请求体

通过`@Body`注解，可以指明一个对象作为HTTP请求体：

```
@POST("users/new")
Call<User> createUser(@Body User user);

Call<User> createUser(@Body RequestBody body);

```

**注意：**HTTP请求体将通过一个转换器(Converter)转换为对应的对象，如果`Retrofit`实例上没有添加任何转换器，则只能使用`RequestBody`类。

### 表单(Form encoded)和多部分(Multipart)请求

可以通过注解，让方法发送`form-encoded`和`multipart`数据。

要发送`form-encoded`数据，在方法上使用`@FormUrlEncoded`注解。每一个键值对是通过参数上的`@Field`注解实现的。

```
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);

```

多部分请求使用`@Multipart`注解，每个部分使用`@Part`注解。

```
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);

```

多部分请求体可以使用`Retrofit`的转换器，也可以自己实现`RequestBody`类来自行处理序列化过程。

### 标头(Header)的处理

可以使用`@Header`注解来设置固定的请求头：

单个：

```
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();

```

多个：

```
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);

```

注意请求头不会互相覆盖，同样名称的请求头会全部包含在请求中。

请求头也可以使用`@Header`注解来动态修改，如果方法参数是`null`，请求头会被忽略，否则，会使用对象的`toString`方法作为请求头的值。

```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)

```

**提示：**每一个请求都需要添加的标头，可以使用[OkHttp拦截器](https://github.com/square/okhttp/wiki/Interceptors)。

### 同步和异步

`Call`对象可以同步执行，也可以异步执行。每个实例只能执行一次，但通过`clone()`方法，能创建一个可执行的新实例。

在Android上，回调会在主线程调用；在JVM上，回调会在执行请求的线程调用。

## Retrofit的配置

`Retrofit`有一个健全的默认配置，同时也提供了灵活的手段来修改配置。

### 转换器

默认情况下，Retrofit只能将HTTP消息头反序列化成OkHttp的`ResponseBody`类型，它的`@Body`注解也只接受`RequestBody`类型。

可以通过添加转换器来支持其他类型。Retrofit提供了如下几个模块，来适配流行的序列化库：

*   **Gson**: com.squareup.retrofit2:converter-gson
*   **Jackson**: com.squareup.retrofit2:converter-jackson
*   **Moshi**: com.squareup.retrofit2:converter-moshi
*   **Protobuf**: com.squareup.retrofit2:converter-protobuf
*   **Wire**: com.squareup.retrofit2:converter-wire
*   **Simple XML**: com.squareup.retrofit2:converter-simplexml
*   **Scalars (primitives, boxed, and String)**: com.squareup.retrofit2:converter-scalars

下面是使用`GsonConverterFactory`的例子，来生成前文中提到的`GitHubService`接口，它使用Gson库来做转换：

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);

```

### 自定义转换器

如果你的服务器API使用了其它的数据格式(例如YAML，txt)，或者针对已有的格式，你想使用不同的库来实现，你可以很容易地创建自己的转换器。创建一个类，继承`Converter.Factory`，在实例化`Retrofit`时传入。