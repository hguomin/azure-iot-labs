---
sidebar_position: 3
---

# 实验三：IoT设备数据的可视化


## 步骤一：添加项目依赖

在注释`<!-- Lab3: data visualization -->`处添加以下依赖：

```xml title=pom.xml
    <!-- Lab3: data visualization -->
    <dependency>
        <groupId>com.microsoft.azure.functions</groupId>
        <artifactId>azure-functions-java-library-signalr</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.11.0</version>
    </dependency>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.9.0</version>
    </dependency>
    <!-- End for Lab3-->
```

## 步骤二：创建SignalR服务（国内Web Pubsub服务未落地）

1. 登录到Azure门户，选择**创建资源**
2. 在**创建资源**页面的**搜索服务和市场**中输入**signalr**，在结果列表中选择**SignalR 服务**
3. 在**SignalR 服务**页面，选择**创建**
4. 在**基本信息**标签页，输入以下信息
5. 点击**审阅+创建**完成SignalR服务的创建

## 步骤三：实现Web页面显示

1. 创建html页面  

    在`src\main`目录下创建`resources\public`目录，并在该目录下创建`index.html`作为Web显示页面，`index.html`内容如下：
    ```html title=src\main\resources\public\index.html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="utf-8" />
            <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
            <meta name="viewport" content="width=device-width, initial-scale=1" />
            <meta name="theme-color" content="#000000" />
            <meta
                name="description"
                content="Azure SignalR web page"
            />
            <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
            <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />

            <title>Azure IoT 动手实验营</title>
        </head>
        <body>
            <h2>Azure IoT 设备遥测</h2>
            <div id="messages"></div>
            
            <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js"></script>
            <script>
                let messages = document.querySelector('#messages');
                const apiBaseUrl = window.location.origin;
                const connection = new signalR.HubConnectionBuilder()
                    .withUrl(apiBaseUrl + '/api')
                    .configureLogging(signalR.LogLevel.Information)
                    .build();
                    connection.on('newMessage', (message) => {
                    document.getElementById("messages").innerHTML = message;
                    });
            
                    connection.start()
                    .catch(console.error);
            </script>
        </body>
    </html>
    ```
2. 添加显示`index.html`的函数  

    在`src\main\java\com\aziot\cloudapp\AzureFunctions.java`类中添加一个名为`index`的`HttpTrigger`Function，代码如下：
    ```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    @FunctionName("index")
    public HttpResponseMessage index(
            @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET, HttpMethod.POST},
                authLevel = AuthorizationLevel.ANONYMOUS)
                HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) throws IOException {
        context.getLogger().info("Java HTTP trigger processed a request.");

        InputStream inputStream = getClass().getClassLoader().getResourceAsStream("public/index.html");
        String text = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());

        return request.createResponseBuilder(HttpStatus.OK).header("Content-Type", "text/html").body(text).build();
    }
    ```
    该函数通过读取`index.html`的内容并返回html响应，从而在浏览器端显示页面内容。

## 实现SignalR的`negotiate`API

SignalR客户端需要通过`negotiate`api获取SignalR服务的连接密钥信息，因此需要实现该函数，在`src\main\java\com\aziot\cloudapp\AzureFunctions.java`类中添加如下代码：
```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    @FunctionName("negotiate")
    public SignalRConnectionInfo negotiate(
            @HttpTrigger(
                name = "req",
                methods = { HttpMethod.POST },
                authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> req,
            @SignalRConnectionInfoInput(
                name = "connectionInfo",
                hubName = "serverless") SignalRConnectionInfo connectionInfo) {

        return connectionInfo;
    }
```
同时需要在上述文件中导入以下类：
```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
import com.microsoft.azure.functions.signalr.SignalRConnectionInfo;
import com.microsoft.azure.functions.signalr.SignalRMessage;
import com.microsoft.azure.functions.signalr.annotation.SignalRConnectionInfoInput;
import com.microsoft.azure.functions.signalr.annotation.SignalROutput;
```

在`local.settings.xml`的`Value`节点中添加键值为**AzureSignalRConnectionString**的配置，将其值设为SignalR服务的连接字符串：
```json title=local.settings.xml
{
  "IsEncrypted": false,
  "Values": {
    ...
    "AzureSignalRConnectionString": "SignalR服务的连接字符串"
  }
}
```

## 实现SignalR消息推送函数

修改`src\main\java\com\aziot\cloudapp\AzureFunctions.java`类中的`receiveIoTHubMessages`函数实现，将收到的IoT Hub设备遥测数据转发到SignalR服务中，代码如下：

```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    /**
     * This function will be invoked when an event is received from Event Hub.
     */
    @FunctionName("ReceiveIoTHubMessages")
    @SignalROutput(name = "$return", hubName = "serverless")
    public SignalRMessage receiveIoTHubMessages(
        @EventHubTrigger(name = "message", eventHubName = "", connection = "IoTHubBuiltinEndpoint", consumerGroup = "$Default", cardinality = Cardinality.ONE) String message,
        final ExecutionContext context
    ) {
        context.getLogger().info("Java Event Hub trigger function executed.");
        context.getLogger().info("Message: " + message);
        return new SignalRMessage("newMessage", message);
    }
```

## 实验效果

执行命令`mvn clean package azure-functions:run`运行程序，并在浏览器中访问`http://localhost:7071/api/index`即可看到设备的遥测数据。

## 参考文档
[Use Java to implement SignalR service](https://learn.microsoft.com/en-us/azure/azure-signalr/signalr-quickstart-azure-functions-java)

