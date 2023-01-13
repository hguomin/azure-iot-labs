---
sidebar_position: 6
---

# 实验六：在应用程序中集成Azure IoT Hub服务端SDKs

在本次实验中，我们将在Azure Functions应用程序中集成Azure IoT Hub的服务端SDK，并通过SDK调用Azure IoT Hub开放的API实现对边缘设备的远程配置、控制管理及应用部署等功能。

## 在应用中引入Azure IoT Hub SDKs

### 步骤一：添加Azure IoT Hub SDKs依赖
在`pom.xml`的`<!-- Lab6: azure iot hub sdk integration -->`注释处添加以下依赖：
```xml title=pom.xml
        <!-- Lab6: azure iot hub sdk integration -->
		<dependency>
			<groupId>com.microsoft.azure.sdk.iot</groupId>
			<artifactId>iot-service-client</artifactId>
			<version>2.0.2</version>
		</dependency>
```

### 步骤二：代码中引入Azure IoT Hub SDKs相关的客户端库

在`src\main\java\com\aziot\cloudapp`目录中新建Spring配置类`AzuIotHubConfiguration`，并复制以下代码到该类文件中：

```java title=src\main\java\com\aziot\cloudapp\AzuIotHubConfiguration.java
package com.aziot.cloudapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.microsoft.azure.sdk.iot.service.auth.IotHubConnectionString;
import com.microsoft.azure.sdk.iot.service.auth.IotHubConnectionStringBuilder;
import com.microsoft.azure.sdk.iot.service.configurations.ConfigurationsClient;
import com.microsoft.azure.sdk.iot.service.methods.DirectMethodsClient;
import com.microsoft.azure.sdk.iot.service.query.QueryClient;
import com.microsoft.azure.sdk.iot.service.registry.RegistryClient;
import com.microsoft.azure.sdk.iot.service.twin.TwinClient;

@Configuration
public class AzuIotHubConfiguration {
    private final String iothubConnectionString = "[Your IoT Hub Connection String]";

    @Bean
    public IotHubConnectionString iotHubConnectionString() {
        return IotHubConnectionStringBuilder.createIotHubConnectionString(this.iothubConnectionString);
    }

    @Bean
    public RegistryClient iotHubRegistryClient() {
        return new RegistryClient(this.iothubConnectionString);
    }

    @Bean
    public TwinClient iotHubTwinClient() {
        return new TwinClient(this.iothubConnectionString);
    }

    @Bean
    public DirectMethodsClient iotHubDirectMethodClient() {
        return new DirectMethodsClient(this.iothubConnectionString);
    }

    @Bean
    public QueryClient iotHubQueryClient() {
        return new QueryClient(this.iothubConnectionString);
    }

    @Bean
    public ConfigurationsClient iotHubConfigurationsClient() {
        return new ConfigurationsClient(this.iothubConnectionString);
    }
}
```
以上代码中导入了调用Azure IoT Hub服务端功能所需的Client API，例如设备注册表、设备孪生、直接方法和查询客户端等，并以Spring Bean对象提供给IoT Hub Service代码进行调用。因此，我们需要创建IoT Hub Service类，并在该类中实现具体的基于IoT Hub服务端的应用功能。

在`src\main\java\com\aziot\cloudapp`目录中新建IoT Hub服务类`AzIotHubService`，并注入Azure IoT Hub SDKs相关的Client，我们将在后续的步骤中逐步添加基于这些Client API的应用功能实现。
```java title=src\main\java\com\aziot\cloudapp\AzIotHubService.java
package com.aziot.cloudapp;

import org.springframework.beans.factory.annotation.Autowired;

import com.microsoft.azure.sdk.iot.service.methods.DirectMethodsClient;
import com.microsoft.azure.sdk.iot.service.query.QueryClient;
import com.microsoft.azure.sdk.iot.service.registry.RegistryClient;
import com.microsoft.azure.sdk.iot.service.twin.TwinClient;

import com.google.gson.JsonArray;

public class AzIotHubService {
    private final RegistryClient registryClient;
    private final TwinClient twinClient;
    private final DirectMethodsClient directMethodsClient;
    private final QueryClient queryClient;

    @Autowired
    public AzIotHubService(RegistryClient registryClient, TwinClient twinClient, DirectMethodsClient directMethodsClient, QueryClient queryClient) {
        this.registryClient = registryClient;
        this.twinClient = twinClient;
        this.directMethodsClient = directMethodsClient;
        this.queryClient = queryClient;
    }

    JsonArray getIoTEdgeDevices() {
        return null;
    }
}

```
至此，我们将可以在`AzIoTHubService`类中实现本次动手实验的核心业务逻辑并在Azure Functions中调用。

## 备注

本实验的代码和lab5合在一起了