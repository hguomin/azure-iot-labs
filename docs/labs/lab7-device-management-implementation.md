---
sidebar_position: 7
---

# 实验七：设备管理功能实现

## 获取设备列表
### 步骤一：获取IoT Hub中的IoT Edge设备列表功能实现

在`src\main\java\com\aziot\cloudapp\AzIotHubService.java`中实现`getIoTEdgeDevices`方法的代码：
```java title=src\main\java\com\aziot\cloudapp\AzIotHubService.java
    public JsonArray getIoTEdgeDevices() {
        JsonArray devicesList = new JsonArray();
        devicesList.add(new JsonObject());

        try {
            Gson gson = new Gson();
            RawQueryResponse query = this.queryClient.queryRaw("SELECT * FROM devices");
            while(query.hasNext()) {
                JsonObject deviceQuery = gson.fromJson(query.next(), JsonObject.class);
                boolean isIoTEdgeDevice = deviceQuery.get("capabilities").getAsJsonObject().get("iotEdge").getAsBoolean();
                if (isIoTEdgeDevice) {
                    String deviceId = deviceQuery.get("deviceId").getAsString();
                    Device device = this.registryClient.getDevice(deviceId);

                    JsonObject deviceJson = new JsonObject();
                    deviceJson.addProperty("deviceId", deviceId);
                    deviceJson.addProperty("primaryKey", device.getPrimaryKey());
                    deviceJson.addProperty("status", device.getStatus().getValue());
                    deviceJson.addProperty("connectionState", device.getConnectionState().getValue());

                    devicesList.add(deviceJson);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return devicesList;
    }
```

### 步骤二：实现获取IoT Edge设备列表的REST API

在`src\main\java\com\aziot\cloudapp\AzureFunctions.java`添加如下代码实现设备列表获取REST API的Azure Function：

```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    //---------Lab7: Get IoT edge devices---------//
    @FunctionName("EdgeDevices")
    public HttpResponseMessage getIoTEdgeDevices(
            @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET},
                authLevel = AuthorizationLevel.ANONYMOUS)
                HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        context.getLogger().info("Get IoT edge devices.");

        return request.createResponseBuilder(HttpStatus.OK).body(handleRequest(context)).build();
    }
```

同时在`src\main\java\com\aziot\cloudapp\AzureIoTApplication.java`类中添加以上函数的响应实现，即调用`AzIotHubService`的`getIoTEdgeDevices`方法获取IoT Edge设备并返回:
```java title=src\main\java\com\aziot\cloudapp\AzureIoTApplication.java
    @Bean(name="EdgeDevices")
    public Supplier<JsonArray> getIoTEdgeDevices(AzIotHubService iotHubService) {
        return () -> {
            return iotHubService.getIoTEdgeDevices();
        };
    }
```

### 步骤三：设备列表获取API测试

在浏览器中访问`http://localhost:7071/api/EdgeDevices`，将返回如下示例结果：
```json
[
  {
    "deviceId": "gm-iotedge-dev1",
    "primaryKey": "lCSUacRLtjdxNk3kzKB57PG3mrulzdCYqT2qE8unY8k=",
    "status": "enabled",
    "connectionState": "Disconnected"
  }
]
```

## IoT Edge模块部署

### 步骤一： AzureFunctions.java
```java title=src\main\java\com\aziot\cloudapp\AzureFunctions.java
    //---------Lab7: Deploy IoT edge module--------//
    @FunctionName("deployIoTEdgeModule")
    public HttpResponseMessage deployIoTEdgeModule(
            @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET, HttpMethod.POST},
                authLevel = AuthorizationLevel.ANONYMOUS)
                HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        context.getLogger().info("Deploy IoT edge modules.");

        return request.createResponseBuilder(HttpStatus.OK).body(handleRequest(request.getBody().get(), context)).build();
        
    }
```

### 步骤二：src\main\java\com\aziot\cloudapp\AzureIoTApplication.java

```java title=src\main\java\com\aziot\cloudapp\AzureIoTApplication.java
    //---------Lab7: Deploy IoT edge module--------//
    @Bean(name = "deployIoTEdgeModule")
    public Function<JsonObject, JsonObject> deployIoTEdgeModule(AzIotHubService iotHubService) {
        return (deploymentSettings) -> {
            return deploymentSettings;
        };
    }
```

### 步骤三：新建src\main\resources\iotedge-deployment-manifest-template.json
```json title=src\main\resources\iotedge-deployment-manifest-template.json

```
