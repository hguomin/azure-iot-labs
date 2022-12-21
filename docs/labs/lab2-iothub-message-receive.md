---
sidebar_position: 2
---
# 实验二：从IoT Hub接收设备数据

## Azure Functions调试的存储账号设置

在Azure Portal创建存储账号

拷贝连接字符串

在local.settings.json中将`AzureWebJobsStorage`的值设为该连接字符串

## 添加IoT Hub的Trigger


在 `src\main\java\com\aziot\cloudapp\AzureFunctions.java`文件中导入`EventHubTrigger`

```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
import com.microsoft.azure.functions.annotation.Cardinality;
import com.microsoft.azure.functions.annotation.EventHubTrigger;

import java.util.List;

```

并在`AzureFunctions`类中添加IoT Hub Trigger的代码

```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    @FunctionName("ReceiveIoTHubMessages")
    public void receiveIoTHubMessages(
        @EventHubTrigger(name = "message", eventHubName = "", connection = "IoTHubBuiltinEndpoint", consumerGroup = "$Default", cardinality = Cardinality.MANY) List<String> message,
        final ExecutionContext context
    ) {
        context.getLogger().info("Java Event Hub trigger function executed.");
        context.getLogger().info("Length:" + message.size());
        message.forEach(singleMessage -> context.getLogger().info(singleMessage));
    }
```

在`local.settings.json`文件中，将`IoTHubBuiltinEndpoint`的值设为您的IoT Hub的内置终结点的连接字符串

```json title=local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "存储账号的连接字符串",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "MAIN_CLASS": "com.aziot.cloudapp.AzureIoTApplication",
    "IoTHubBuiltinEndpoint": "IoT Hub内置终结点的连接字符串"
  }
}
```

启动设备发送，并执行命令`mvn clean package azure-functions:run`运行程序，可以看到接收到设备数据的日志输出如下：
```bash
[2022-12-21T09:19:25.078Z] Length:1
[2022-12-21T09:19:27.076Z] Executing 'Functions.IoTHubMessagesReceiver' (Reason='(null)', Id=72e144b6-c4e2-46a1-92d5-6ac3881b0453)
[2022-12-21T09:19:27.079Z] Trigger Details: PartionId: 1, Offset: 425201780360-425201780360, EnqueueTimeUtc: 2022-12-21T09:19:27.4440000+00:00-2022-12-21T09:19:27.4440000+00:00, SequenceNumber: 50811-50811, Count: 1
[2022-12-21T09:19:27.084Z] 2022-12-21 17:19:27.083  INFO 3940 --- [pool-2-thread-4] o.s.c.function.utils.FunctionClassUtils  : Main class: class com.aziot.cloudapp.AzureIoTApplication
[2022-12-21T09:19:27.085Z] Function "IoTHubMessagesReceiver" (Id: 72e144b6-c4e2-46a1-92d5-6ac3881b0453) invoked by Java Worker
[2022-12-21T09:19:27.085Z] {"messageId":38,"deviceId":"EnvSensor1","temperature":29.696880787026252,"humidity":79.76763248474552}
[2022-12-21T09:19:27.085Z] Java Event Hub trigger function executed.
[2022-12-21T09:19:27.085Z] Executed 'Functions.IoTHubMessagesReceiver' (Succeeded, Id=72e144b6-c4e2-46a1-92d5-6ac3881b0453, Duration=9ms)
```
## 参考文档

[Connect functions to Azure services using bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-iot-trigger?tabs=in-process%2Cfunctionsv2%2Cextensionv5&pivots=programming-language-java)
