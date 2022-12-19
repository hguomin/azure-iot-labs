---
sidebar_position: 1
---

# 实验一：创建动手实验的应用程序项目

在本次实验中，我们将创建并初始化实验所需的各个应用程序项目的代码。

## 创建云端应用程序项目

云端应用程序是一个Azure Functions项目，我们将在该项目中通过实现具体的Function来完成整个应用程序功能的开发。打开命令行程序，按以下步骤完成本次实验。

### 步骤一：创建基本的Azure Functions项目
在命令行程序中，输入并执行以下Maven命令创建Azure Functions的Java应用程序项目：
```bash
mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=11"
```
按照Maven命令行的提示输入以下信息：

| 命令提示         | 输入值             |
| -----------     | ----------         |
| `groupId`       | 应用程序Group Id, 输入：`com.aziot.cloudapp` |
| `artifactId`    | 应用程序Artifact Id，输入：`aziot-lab-cloudapp` |
| `version`    | 应用程序版本，按回车确认 |
| `package`    | 此处需要输入应用程序打包信息，包括Azure Functions部署的配置，按回车键直接跳过，应用部署前再进行配置    |

按回车键确认，这样一个Azure Functions项目就创建完成了。

在命令行中进入到项目根目录，执行以下命令运行程序：
```bash
cd aziot-lab-cloudapp
mvn package azure-functions:run
```

运行成功后在命令行控制台中将输出以下内容：
```bash
[INFO] Azure Functions Core Tools found.

Azure Functions Core Tools
Core Tools Version:       4.0.4915 Commit hash: N/A  (64-bit)
Function Runtime Version: 4.14.0.19631

Functions:

        HttpExample: [GET,POST] http://localhost:7071/api/HttpExample

For detailed output, run func with --verbose flag.
[2022-12-19T12:22:47.293Z] OpenJDK 64-Bit Server VM warning: Options -Xverify:none and -noverify were deprecated in JDK 13 and will likely be removed in a future release.
[2022-12-19T12:22:51.407Z] Worker process started and initialized.
[2022-12-19T12:22:52.082Z] Host lock lease acquired by instance ID '00000000000000000000000071C048FA'.
```

打开浏览时访问`http://localhost:7071/api/HttpExample?name=World`时将显示`Hello, World`。

### 步骤二：为Azure Functions项目添加Spring Boot和Spring Cloud Function支持

在云端应用的实现中，我们将借助Spring Boot的依赖注入机制和[Spring Cloud Function](https://spring.io/projects/spring-cloud-function)的Serverless编程抽象来简化程序功能的开发。

#### 项目的`pom.xml`配置

打开项目的`pom.xml`文件，按以下步骤进行配置：

1. 在`<modelVersion />`和`<groupId />`节点之间添加Spring Boot的Parent POM。
    ```xml title=pom.xml
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.6.6</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
    ```
2. 在`<dependencies>...<dependencies/>`节点中添加Spring Boot的测试依赖。
    ```xml title=pom.xml
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
    ```
3. 在`<build><plugins>...<plugins/><build/>`节点中添加Spring Boot的构建插件。
    ```xml title=pom.xml
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
    ```

4. 在`<project>...</project>`节点中添加Spring Cloud Function和Azure Functions的依赖管理。
    ```xml title=pom.xml
        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-function-dependencies</artifactId>
                    <version>3.2.3</version>
                    <type>pom</type>
                    <scope>import</scope>
                </dependency>
                <dependency>
                    <groupId>com.microsoft.azure.functions</groupId>
                    <artifactId>azure-functions-java-library</artifactId>
                    <version>${azure.functions.java.library.version}</version>
                </dependency>
            </dependencies>
        </dependencyManagement>
    ```
5. 添加Spring Cloud Function和Azure Functions的依赖    
   
   首先删除`<dependencies>...</dependencies>`中的以下内容：

   ```xml title=pom.xml
            <dependency>
                <groupId>com.microsoft.azure.functions</groupId>
                <artifactId>azure-functions-java-library</artifactId>
                <version>${azure.functions.java.library.version}</version>
            </dependency>
    ```  

    并替换为Spring Cloud Function和Azure Functions的依赖：
    ```xml title=pom.xml
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-function-web</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-function-adapter-azure</artifactId>
            </dependency>
    ```

#### 基于Spring Cloud Function实现项目中的函数功能  

1. 基于Spring Cloud Function修改默认的函数实现  

    为防止类名与Java的Function重名，将`src\main\java\com\aziot\cloudapp\Function.java`的文件名修改为`src\main\java\com\aziot\cloudapp\AzureFunctions.java`，同时将文件中的`Function`类实现改为如下代码：
    ```java title=src\main\java\com\aziot\cloudapp\AzureFunction.java   
    import org.springframework.cloud.function.adapter.azure.FunctionInvoker;
    /**
    * Azure Functions with HTTP Trigger.
    */
    public class AzureFunctions extends FunctionInvoker<String, String> {
        /**
        * This function listens at endpoint "/api/HttpExample". Two ways to invoke it using "curl" command in bash:
        * 1. curl -d "HTTP Body" {your host}/api/HttpExample
        * 2. curl "{your host}/api/HttpExample?name=HTTP%20Query"
        */
        @FunctionName("HttpExample")
        public HttpResponseMessage run(
                @HttpTrigger(
                    name = "req",
                    methods = {HttpMethod.GET, HttpMethod.POST},
                    authLevel = AuthorizationLevel.ANONYMOUS)
                    HttpRequestMessage<Optional<String>> request,
                final ExecutionContext context) {
            context.getLogger().info("Java HTTP trigger processed a request.");

            // Parse query parameter
            final String query = request.getQueryParameters().get("name");
            final String name = request.getBody().orElse(query);

            if (name == null) {
                return request.createResponseBuilder(HttpStatus.BAD_REQUEST).body("Please pass a name on the query string or in the request body").build();
            } else {
                return request.createResponseBuilder(HttpStatus.OK).body(handleRequest(name, context)).build();
            }
        }
    }
    ```

2. 添加Spring Boot的主类  

    在代码目录`src\main\java\com\aziot\cloudapp`中创建`AzureIoTApplication.java`文件，文件中的代码内容如下：  

    ```java title=src\main\java\com\aziot\cloudapp\AzureIoTApplication.java
    package com.aziot.cloudapp;

    import java.util.function.Function;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.annotation.Bean;

    @SpringBootApplication
    public class AzureIoTApplication {
        public AzureIoTApplication() {
            System.out.println("AzureIoTApplication is created.");
        }
        public static void main(String[] args) {
            SpringApplication.run(AzureIoTApplication.class, args);
        }

        @Bean(name="HttpExample")
        public Function<String, String> httpExample() {
            return name -> {
                return "Hello " + name;
            };
        }
    }
    ```
#### 配置Azure Functions的函数入口主类

在项目的`local.settings.json`添加项目的入口主类，添加后`local.settings.json`的完整内容如下：
```json title=local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "MAIN_CLASS":"com.aziot.cloudapp.AzureIoTApplication"
  }
}
```

#### 删除单元测试代码

为简化实验过程，我们将删除所有单元测试代码，将目录`src\test`中的内容全部删除即可。

#### 运行项目应用程序
通过以上配置，我们在项目中为Azure Functions添加了Spring Boot和Spring Cloud Function的支持，再次执行命令
```bash 
mvn package azure-functions:run
```
运行应用并访问`HttpExample`的API，可以看到Spring Boot输出的启动信息

```bash
[2022-12-19T13:54:33.995Z] 21:54:31.363 [pool-2-thread-2] INFO org.springframework.cloud.function.adapter.azure.FunctionInvoker - Initializing: class org.springframework.cloud.function.web.RestApplication
[2022-12-19T13:54:33.996Z]   .   ____          _            __ _ _
[2022-12-19T13:54:33.998Z]  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
[2022-12-19T13:54:33.999Z] ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
[2022-12-19T13:54:34.001Z]  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
[2022-12-19T13:54:34.002Z]   '  |____| .__|_| |_|_| |_\__, | / / / /
[2022-12-19T13:54:34.004Z]  =========|_|==============|___/=/_/_/_/
[2022-12-19T13:54:34.005Z]  :: Spring Boot ::                (v2.6.6)
[2022-12-19T13:54:34.007Z] 2022-12-19 21:54:31.780  INFO 30168 --- [pool-2-thread-2] o.s.boot.SpringApplication
      : Starting application using Java 17.0.5 on CN-GUHUANG-P1 with PID 30168 (D:\Microsoft\Projects\IoT\aziot-lab-iiot-e2c\cloud-app\aziot-cloudapp\target\azure-functions\aziot-cloudapp-20221219175736561\lib\spring-cloud-function-web-3.2.3.jar started by guhuang in D:\Microsoft\Projects\IoT\aziot-lab-iiot-e2c\cloud-app\aziot-cloudapp\target\azure-functions\aziot-cloudapp-20221219175736561)
[2022-12-19T13:54:34.009Z] 2022-12-19 21:54:31.782  INFO 30168 --- [pool-2-thread-2] o.s.boot.SpringApplication
      : No active profile set, falling back to 1 default profile: "default"
[2022-12-19T13:54:34.011Z] 2022-12-19 21:54:32.700  INFO 30168 --- [pool-2-thread-2] o.s.boot.SpringApplication
      : Started application in 1.257 seconds (JVM running for 140.428)
```

## 创建Azure IoT Edge模块项目