#### 1.Retrofit的构建 {#421-retrofit的构建}

这里依然是通过构造者模式进行构建retrofit对象，好在其内部的成员变量比较少，我们直接看build\(\)方法。

```java
public Builder() {
    this(Platform.get());
}

public Retrofit build() {
  if (baseUrl == null) {
    throw new IllegalStateException("Base URL required.");
  }

  okhttp3.Call.Factory callFactory = this.callFactory;
  if (callFactory == null) {
    callFactory = new OkHttpClient();
  }

  Executor callbackExecutor = this.callbackExecutor;
  if (callbackExecutor == null) {
    callbackExecutor = platform.defaultCallbackExecutor();
  }

  // Make a defensive copy of the adapters and add the default Call adapter.
  List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
  adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

  // Make a defensive copy of the converters.
  List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

  return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
      callbackExecutor, validateEagerly);
}
```

baseUrl必须指定，这个是理所当然的；

* 然后可以看到如果不着急设置callFactory，则默认直接`new OkHttpClient()`，可见如果你需要对okhttpclient进行详细的设置，需要构建`OkHttpClient`对象，然后传入；
* 接下来是callbackExecutor，这个想一想大概是用来将回调传递到UI线程了，当然这里设计的比较巧妙，利用platform对象，对平台进行判断，判断主要是利用`Class.forName("")`进行查找，具体代码已经被放到文末，如果是Android平台，会自定义一个
  `Executor`对象，并且利用`Looper.getMainLooper()`实例化一个handler对象，在`Executor`内部通过`handler.post(runnable)`
  ，ok，整理凭大脑应该能构思出来，暂不贴代码了。

* 接下来是adapterFactories，这个对象主要用于对Call进行转化，基本上不需要我们自己去自定义。
* 最后是converterFactories，该对象用于转化数据，例如将返回的`responseBody`转化为对象等；当然不仅仅是针对返回的数据，还能用于一般备注解的参数的转化例如`@Body`标识的对象做一些操作，后面遇到源码详细再描述。

ok，总体就这几个对象去构造retrofit，还算比较少的~~

#### 2 具体Call构建流程 {#422-具体call构建流程}

我们构造完成retrofit，就可以利用retrofit.create方法去构建接口的实例了，上面我们已经分析了这个环节利用了动态代理，而且我们也分析了具体的Call的构建流程在invoke方法中，下面看代码：

```
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    //...
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
           @Override 
          public Object invoke(Object proxy, Method method, Object... args){
            //...
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
}
```

主要也就三行代码，第一行是根据我们的method将其包装成ServiceMethod，第二行是通过ServiceMethod和方法的参数构造`retrofit2.OkHttpCall`对象，第三行是通过`serviceMethod.callAdapter.adapt()`方法，将`OkHttpCall`进行代理包装；

下面一个一个介绍：

* ServiceMethod应该是最复杂的一个类了，包含了将一个method转化为Call的所有的信息。

```java
#Retrofit class
ServiceMethod loadServiceMethod(Method method) {
    ServiceMethod result;
    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }

#ServiceMethod
public ServiceMethod build() {
      callAdapter = createCallAdapter();
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter();

      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }


      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p];
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }

        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }

        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      return new ServiceMethod<>(this);
}
```

直接看build方法，首先拿到这个callAdapter最终拿到的是我们在构建retrofit里面时adapterFactories时添加的，即为：`new ExecutorCallbackCall<>(callbackExecutor, call)`，该`ExecutorCallbackCall`唯一做的事情就是将原本call的回调转发至UI线程。

接下来通过`callAdapter.responseType()`返回的是我们方法的实际类型，例如:`Call<User>`,则返回`User`类型，然后对该类型进行判断。

接下来是`createResponseConverter`拿到responseConverter对象，其当然也是根据我们构建retrofit时,`addConverterFactory`添加的ConverterFactory对象来寻找一个合适的返回，寻找的依据主要看该converter能否处理你编写方法的返回值类型，默认实现为`BuiltInConverters`，仅仅支持返回值的实际类型为`ResponseBody`和`Void`，也就说明了默认情况下，是不支持`Call<User>`这类类型的。

接下来就是对注解进行解析了，主要是对方法上的注解进行解析，那么可以拿到httpMethod以及初步的url（包含占位符）。

后面是对方法中参数中的注解进行解析，这一步会拿到很多的`ParameterHandler`对象，该对象在`toRequest()`构造Request的时候调用其apply方法。

ok，这里我们并没有去一行一行查看代码，其实意义也不太大，只要知道ServiceMethod主要用于将我们`接口中的方法`转化为一个`Request对象`，于是根据我们的接口返回值确定了responseConverter,解析我们方法上的注解拿到初步的url,解析我们参数上的注解拿到构建RequestBody所需的各种信息，最终调用toRequest的方法完成Request的构建。

* 接下来看OkHttpCall的构建，构造函数仅仅是简单的赋值

```
OkHttpCall(ServiceMethod<T> serviceMethod, Object[] args) {
    this.serviceMethod = serviceMethod;
    this.args = args;
  }
```

* 最后一步是serviceMethod.callAdapter.adapt\(okHttpCall\)

我们已经确定这个callAdapter是`ExecutorCallAdapterFactory.get()`

对应代码为：

```java
final class ExecutorCallAdapterFactory extends CallAdapter.Factory {
  final Executor callbackExecutor;

  ExecutorCallAdapterFactory(Executor callbackExecutor) {
    this.callbackExecutor = callbackExecutor;
  }

  @Override
  public CallAdapter<Call<?>> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public <R> Call<R> adapt(Call<R> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }
```

可以看到`adapt`返回的是`ExecutorCallbackCall`对象，继续往下看：

```
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      if (callback == null) throw new NullPointerException("callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
    @Override public Response<T> execute() throws IOException {
      return delegate.execute();
    }
  }
```

可以看出ExecutorCallbackCall仅仅是对Call对象进行封装，类似装饰者模式，只不过将其执行时的回调通过callbackExecutor进行回调到UI线程中去了。

#### 3 执行Call {#423-执行call}

在4.2.2我们已经拿到了经过封装的`ExecutorCallbackCall`类型的call对象，实际上就是我们实际在写代码时拿到的call对象，那么我们一般会执行`enqueue`方法，看看源码是怎么做的

首先是`ExecutorCallbackCall.enqueue`方法，代码在4.2.2，可以看到除了将onResponse和onFailure回调到UI线程，主要的操作还是delegate完成的，这个delegate实际上就是OkHttpCall对象，我们看它的enqueue方法

```
@Override
public void enqueue(final Callback<T> callback)
{
    okhttp3.Call call;
    Throwable failure;

    synchronized (this)
    {
        if (executed) throw new IllegalStateException("Already executed.");
        executed = true;

        try
        {
            call = rawCall = createRawCall();
        } catch (Throwable t)
        {
            failure = creationFailure = t;
        }
    }

    if (failure != null)
    {
        callback.onFailure(this, failure);
        return;
    }

    if (canceled)
    {
        call.cancel();
    }

    call.enqueue(new okhttp3.Callback()
    {
        @Override
        public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
                throws IOException
        {
            Response<T> response;
            try
            {
                response = parseResponse(rawResponse);
            } catch (Throwable e)
            {
                callFailure(e);
                return;
            }
            callSuccess(response);
        }

        @Override
        public void onFailure(okhttp3.Call call, IOException e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callFailure(Throwable e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callSuccess(Response<T> response)
        {
            try
            {
                callback.onResponse(OkHttpCall.this, response);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }
    });
}
```

没有任何神奇的地方，内部实际上就是okhttp的Call对象，也是调用`okhttp3.Call.enqueue`方法。

中间对于`okhttp3.Call`的创建代码为：

```
private okhttp3.Call createRawCall() throws IOException
{
    Request request = serviceMethod.toRequest(args);
    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
    if (call == null)
    {
        throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}
```

可以看到，通过`serviceMethod.toRequest`完成对request的构建，通过request去构造call对象，然后返回.

中间还涉及一个`parseResponse`方法，如果顺利的话，执行的代码如下：

```
Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException
{
    ResponseBody rawBody = rawResponse.body();
    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);

    T body = serviceMethod.toResponse(catchingBody);
    return Response.success(body, rawResponse);
```

通过serviceMethod对ResponseBody进行转化，然后返回，转化实际上就是通过`responseConverter`的convert方法。

```
#ServiceMethod
 T toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }
```

ok，关于`responseConverter`后面还会细说，不用担心。

到这里，我们整个源码的流程分析就差不多了，目的就掌握一个大体的原理和执行流程，了解下几个核心的类。

那么总结一下：

* 首先构造retrofit，几个核心的参数呢，主要就是baseurl,callFactory\(默认okhttpclient\),converterFactories,adapterFactories,excallbackExecutor。
* 然后通过create方法拿到接口的实现类，这里利用Java的`Proxy`类完成动态代理的相关代理
* 在invoke方法内部，拿到我们所声明的注解以及实参等，构造ServiceMethod，ServiceMethod中解析了大量的信息，最痛可以通过`toRequest`构造出`okhttp3.Request`对象。有了`okhttp3.Request`对象就可以很自然的构建出`okhttp3.call`，最后calladapter对Call进行装饰返回。
* 拿到Call就可以执行enqueue或者execute方法了

ok,了解这么多足以。

