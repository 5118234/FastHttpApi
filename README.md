`BeetleX.FastHttpApi` is a lightweight and high-performance HTTP service component in the dotnet core platform that supports WebSocket and SSL.

<a href="https://www.nuget.org/packages/BeetleX.FastHttpApi/" target="_blank"> 
    <img src="https://img.shields.io/nuget/vpre/beetlex.fasthttpapi?label=FastHttpApi"> 
							  <img src="https://img.shields.io/nuget/dt/BeetleX.FastHttpApi">
							  </a>
                              
## samples
[**[https://github.com/IKende/BeetleX-Samples](https://github.com/IKende/BeetleX-Samples)**]

## [TechEmpower Framework Benchmarks for .net (2019-8-9)](https://www.techempower.com/benchmarks/#section=test&runid=01ac057d-9871-4f71-9ed4-8320dcb8fdca)
- Json
![](https://img3.tomap.me/json.png)
- Single query
![](https://img3.tomap.me/db.png)
- Multiple queries 
![](https://img3.tomap.me/query.png)
- Fortunes
![](https://img3.tomap.me/fortune.png)

## Using

### Install BeetleX.FastHttpApi
`Install-Package BeetleX.FastHttpApi`

### Base sample code
``` csharp
    [Controller]
    class Program
    {
        private static BeetleX.FastHttpApi.HttpApiServer mApiServer;
        static void Main(string[] args)
        {
            mApiServer = new BeetleX.FastHttpApi.HttpApiServer();
            mApiServer.Options.LogLevel = BeetleX.EventArgs.LogType.Trace;
            mApiServer.Options.LogToConsole = true;
            mApiServer.Debug();//set view path with vs project folder
            mApiServer.Register(typeof(Program).Assembly);
            //mApiServer.Options.Port=80; set listen port to 80
            mApiServer.Open();//default listen port 9090  
            Console.Write(mApiServer.BaseServer);
            Console.Read();
        }
        // Get /hello?name=henry 
        // or
        // Get /hello/henry
        [Get(Route="{name}")]
        public object Hello(string name)
        {
            return $"hello {name} {DateTime.Now}";
        }
        // Get /GetTime  
        public object GetTime()
        {
            return DateTime.Now;
        }
    }
```
### Hosting service
`Install-Package BeetleX.FastHttpApi.Hosting -Version 0.8.2`
``` csharp
     var builder = new HostBuilder()
     .ConfigureServices((hostContext, services) =>
      {
              services
               .AddSingleton<UserService>()
               .UseBeetlexHttp(o => {
                        o.Port = 8080;
                        o.LogToConsole = true;
                        o.LogLevel = BeetleX.EventArgs.LogType.Debug;
                        o.SetDebug();
               }, typeof(Program).Assembly);
        });
       builder.Build().Run();
```

## Setting https
- HttpConfig.json
```
 "SSL": true,
 "CertificateFile": "you.com.pfx",
 "CertificatePassword": "******",
```
- Code
``` csharp
mApiServer.ServerConfig.SSL=true;
mApiServer.ServerConfig.CertificateFile="you.com.pfx";
mApiServer.ServerConfig.CertificatePassword="******";
```
## Defined result
- Text result
``` csharp
    public class TextResult : ResultBase
    {
        public TextResult(string text)
        {
            Text = text == null ? "" : text;
        }
        public string Text { get; set; }
        public override bool HasBody => true;
        public override void Write(PipeStream stream, HttpResponse response)
        {
            stream.Write(Text);
        }
    }
```
- Use result
``` csharp
        public object plaintext()
        {
            return new TextResult("Hello, World!");
        }
```
## Cookies 
``` csharp
        public object SetCookie(string name, string value, IHttpContext context)
        {
            Console.WriteLine(context.Data);
            context.Response.SetCookie(name, value);
            return $"{DateTime.Now}{name}={value}";
        }

        public string GetCookie(string name, IHttpContext context)
        {
            Console.WriteLine(context.Data);
            return $"{DateTime.Now} {name}= {context.Request.Cookies[name]}";
        }
```
## Header
``` csharp
        public void SetHeader(string token,IHttpContext context)
        {
            context.Response.Header["Token"]=token;
        }

        public string GetHeader(string name, IHttpContext context)
        {
            return  context.Request.Header[name];
        }
```
## Data bind
- Url

`/hello?name=xxx`or`/hello/henry`
``` csharp
        [Get(Route = "{name}")]
        public object Hello(string name, IHttpContext context)
        {
            return $"hello {name} {DateTime.Now}";
        }
```
`/SetValue?id=xxx&value=xxxx`or`/SetValue/xxx-xxx`
``` csharp
        [Get(Route = "{id}-{value}")]
        public object SetValue(string id, string value, IHttpContext context)
        {
            return $"{id}={value} {DateTime.Now}";
        }
```
- Json

`{"name":"xxxx","value":"xxx"}`
``` csharp
        [Post]
        [JsonDataConvert]
        public object Post(string name, string value, IHttpContext context)
        {
            Console.WriteLine(context.Data);
            return $"{name}={value}";
        }
```
or
``` csharp
        [Post]
        [JsonDataConvert]
        public object Post(Property body, IHttpContext context)
        {
            Console.WriteLine(context.Data);
            return $"{body.name}={body.value}";
        }
```
- x-www-form-urlencoded

`name=aaa&value=aaa`
``` csharp
        [Post]
        [FormUrlDataConvert]
        public object PostForm(string name, string value, IHttpContext context)
        {
            Console.WriteLine(context.Data);
            return $"{name}={value}";
        }
```
- multipart/form-data
``` csharp
        [Post]
        [MultiDataConvert]
        public object UploadFile(string remark, IHttpContext context)
        {
            foreach (var file in context.Request.Files)
                using (System.IO.Stream stream = System.IO.File.Create(file.FileName))
                {
                    file.Data.CopyTo(stream);
                }
            return $"{DateTime.Now} {remark} {string.Join(",", (from fs in context.Request.Files select fs.FileName).ToArray())}";
        }
```
- Read stream
``` csharp
        [Post]
        [NoDataConvert]
        public object PostStream(IHttpContext context)
        {
            Console.WriteLine(context.Data);
            string value = context.Request.Stream.ReadString(context.Request.Length);
            return value;
        }
```
## Filter
- Defined filter
``` csharp
    public class GlobalFilter : FilterAttribute
    {
        public override bool Executing(ActionContext context)
        {
            Console.WriteLine(DateTime.Now + " globalFilter execting...");
            return base.Executing(context);
        }
        public override void Executed(ActionContext context)
        {
            base.Executed(context);
            Console.WriteLine(DateTime.Now + " globalFilter executed");
        }
    }
```
- Use
``` csharp
        [CustomFilter]
        public string Hello(string name)
        {
            return DateTime.Now + " hello " + name;
        }
```
or
``` csharp
    [Controller]
    [CustomFilter]
    public class ControllerTest
    {
    
    }
```
- Skip filter
``` csharp
        [SkipFilter(typeof(GlobalFilter))]
        public string Hello(string name)
        {
            return DateTime.Now + " hello " + name;
        }
```
## Parameters validation
``` csharp
public bool Register(
      [StringRegion(Min = 5)]string name,
      [StringRegion(Min = 5)]string pwd,
      [DateRegion(Min = "2019-1-1", Max = "2019-2-1")]DateTime dateTime,
      [EmailFormater]string email,
      [IPFormater]string ipaddress,
      [NumberRegion(Min = 18, Max = 56)]int age,
      [DoubleRegion(Min = 10)]double memory
                  )
        {
           return true;
        }
```
## Async action
``` csharp
        [Get(Route = "{name}")]
        public Task<String> Hello(string name)
        {
            string result = $"hello {name} {DateTime.Now}";
            return Task.FromResult(result);
        }

        public async Task<String> Wait()
        {
            await Task.Delay(2000);
            return $"{DateTime.Now}";
        }
```
## Cross domain
``` csharp
        [Options(AllowOrigin = "www.ikende.com")]
        public string GetTime(IHttpContext context)
        {
            return DateTime.Now.ToShortDateString();
        }
```
## Websocket
- Server
``` csharp
[Controller]
    class Program
    {
        private static BeetleX.FastHttpApi.HttpApiServer mApiServer;
        static void Main(string[] args)
        {
            mApiServer = new BeetleX.FastHttpApi.HttpApiServer();
            mApiServer.Debug();
            mApiServer.Register(typeof(Program).Assembly);
            mApiServer.Open();
            Console.Write(mApiServer.BaseServer);
            Console.Read();
        }
        // Get /hello?name=henry 
        // or
        // Get /hello/henry
        [Get(R"{name}")]
        public object Hello(string name)
        {
            return $"hello {name} {DateTime.Now}";
        }
        // Get /GetTime  
        public object GetTime()
        {
            return DateTime.Now;
        }
    }
```
- Hello

Request json
``` javascript
{
      url: '/Hello', 
      params: { name: 'test' }
}
```
- GetTime

Request json
``` javascript
{
      url: '/GetTime', 
      params: { }
}
```
## Using javascript VSIX
download https://github.com/IKende/FastHttpApi/blob/master/Extend/FastHttpApi.JSCreater.zip
unzip execute `FastHttpApi.JSCreaterVSIX.vsix` install to vs(support only vs2017),
change controller file custom tool property to 'JSAPI'
