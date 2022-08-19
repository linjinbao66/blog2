---
title: 华为云sdk踩坑
date: 2021-08-06
---

## 简介

工作中用到了华为云，想使用华为云的sdk实现批量启动流水线，分发代码等任务，简单试用了下，发现问题太多。

## 问题

1. api区域开放问题

   华为云的api调用大致有3种方式，apiexplorer在线调用、hcloud命令行调用，以及通过sdk调用。而同时华为云有区域的区分，比如华东-上海二，华东-北京一这种。api在有些区域开放了调用，但是在另一区域则不开放。举例说明：CloudPipeline（流水线）api开放区域为：上海二、上海一、北京一、北京四、广州，一共5个区域，但是CloudBuild（编译构建）api开放区域则缺少了上海一。要知道CloudBuild会依赖CloudPipeline使用，前一步的api开放了，后一步却不开放，导致根本无法使用。

2. 文档没有示例或者示例是错误的

   CloudPipeline的一个接口，**ListPipelineSimpleInfo**，官方给出的请求示例为：

   ```code
   "POST https://{endpoint}/v3/pipelines/list"
   ```

   文档中根本没给出这个endpoint是什么样式的，后续经过尝试发现hcloud也会配置这个endpoint，其为`cn-east-3`，但是sdk调用时，其格式变成了：`cloudpipeline-ext.cn-east-3.myhuaweicloud.com`。样例的缺失，导致了调试成本巨大。

3. api混乱，缺少

   举例来说，CloudPipeline流水线api，没有查看单条流水线详情的api等等

## 收获

经过不断的踩坑，实验出来了几个api的调用示例，贴出来以后备用。

pom.xml

```xml
<dependencies>
  <!-- https://mvnrepository.com/artifact/com.huaweicloud.sdk/huaweicloud-sdk-cloudpipeline -->
  <dependency>
    <groupId>com.huaweicloud.sdk</groupId>
    <artifactId>huaweicloud-sdk-cloudpipeline</artifactId>
    <version>3.0.54</version>
  </dependency>
  <dependency>
    <groupId>com.huaweicloud.sdk</groupId>
    <artifactId>huaweicloud-sdk-cloudbuild</artifactId>
    <version>3.0.54</version>
</dependency>
```

Test1.java

```java
package pipline;


import com.huaweicloud.sdk.cloudpipeline.v2.CloudPipelineClient;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartNewPipelineRequest;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartNewPipelineResponse;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartPipelineBuildParams;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartPipelineParameters;
import com.huaweicloud.sdk.core.auth.BasicCredentials;
import com.huaweicloud.sdk.core.http.HttpConfig;

public class Test1 {

    public static void main(String[] args) {
        String ak = "XTSUIG9JFXG***S7AWP";
        String sk = "01KQni1q9rM***De6flqxGQf2XxgHYW";
        String endpoint = "cloudpipeline-ext.cn-east-3.myhuaweicloud.com";
        String projectId = "0a9b301b9a***8537a4cb";

        // 配置客户端属性
        HttpConfig config = HttpConfig.getDefaultHttpConfig();
        config.withIgnoreSSLVerification(true);

        // 创建认证
        BasicCredentials auth = new BasicCredentials()
                .withAk(ak)
                .withSk(sk)
                .withProjectId(projectId);

        CloudPipelineClient cloudPipelineClient = CloudPipelineClient.newBuilder()
                .withEndpoint(endpoint)
                .withHttpConfig(config)
                .withCredential(auth)
                .build();

        StartPipelineParameters body = new StartPipelineParameters().addBuildParamsItem(
                new StartPipelineBuildParams().withName("repoLabel").withValue("king***-1-5.1.2")
                .withName("codeBranch").withValue("master")
                .withName("gitUrl").withValue("git@cod***01/test02.git")
                .withName("tag").withValue("ki**.2-1-5.1.2")
        );


        StartNewPipelineRequest startNewPipelineRequest = new StartNewPipelineRequest().withPipelineId("14086662039**2b275a6afc")
                .withBody(body);


        StartNewPipelineResponse startNewPipelineResponse = cloudPipelineClient.startNewPipeline(startNewPipelineRequest);

        System.out.println(startNewPipelineResponse);
    }
}
```

Test1运行结果：

```shell
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
class StartNewPipelineResponse {
    pipelineId: 14086662039**2b275a6afc
    buildId: 29
}

Process finished with exit code 0
```

其启动了流水线，参数中gitUrl和tag并未生效，问题无解

Test2.java

```java
package cloudbuild;


import com.huaweicloud.sdk.cloudbuild.v3.CloudBuildClient;
import com.huaweicloud.sdk.cloudbuild.v3.model.*;
import com.huaweicloud.sdk.cloudpipeline.v2.CloudPipelineClient;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartNewPipelineRequest;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartNewPipelineResponse;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartPipelineBuildParams;
import com.huaweicloud.sdk.cloudpipeline.v2.model.StartPipelineParameters;
import com.huaweicloud.sdk.core.auth.BasicCredentials;
import com.huaweicloud.sdk.core.http.HttpConfig;

public class Test2 {

    public static void main(String[] args) {
        String ak = "XTSUIG9JFXG***S7AWP";
        String sk = "01KQni1q9rM***De6flqxGQf2XxgHYW";
        String endpoint = "cloudpipeline-ext.cn-east-3.myhuaweicloud.com";
        String projectId = "0a9b301b9a***8537a4cb";

        // 配置客户端属性
        HttpConfig config = HttpConfig.getDefaultHttpConfig();
        config.withIgnoreSSLVerification(true);

        // 创建认证
        BasicCredentials auth = new BasicCredentials()
                .withAk(ak)
                .withSk(sk)
                .withProjectId(projectId);

        CloudBuildClient cloudBuildClient = CloudBuildClient.newBuilder()
                .withEndpoint(endpoint)
                .withHttpConfig(config)
                .withCredential(auth)
                .build();

//        Scm scm = new Scm()
//                .withBuildTag("ki***.1.2");
//
//        RunJobRequestBody body = new RunJobRequestBody()
//                .withJobId()
//                .withScm(scm)

        ShowJobListByProjectIdRequest showJobListByProjectIdRequest = new ShowJobListByProjectIdRequest()
                .withProjectId("0a9b301b9a***8537a4cb").withPageIndex(1).withPageSize(10);


        ShowJobListByProjectIdResponse showJobListByProjectIdResponse = cloudBuildClient.showJobListByProjectId(showJobListByProjectIdRequest);
        System.out.println(showJobListByProjectIdResponse);
    }
}
```

Test2运行结果：

```shell
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Exception in thread "main" ClientRequestException {
    httpStatusCode: 404
    errorCode: APIGW.0101
    errorMsg: The API does not exist or has not been published in the environment
    requestId: 26c1db3b8c3f6a0e359e347d49299cfd
}
 at com.huaweicloud.sdk.core.exception.ServiceResponseException.mapException(ServiceResponseException.java:85)
 at com.huaweicloud.sdk.core.exception.ServiceResponseException.mapException(ServiceResponseException.java:73)
 at com.huaweicloud.sdk.core.HcClient.handleException(HcClient.java:328)
 at com.huaweicloud.sdk.core.HcClient.syncInvokeHttp(HcClient.java:172)
 at com.huaweicloud.sdk.core.HcClient.syncInvokeHttp(HcClient.java:141)
 at com.huaweicloud.sdk.cloudbuild.v3.CloudBuildClient.showJobListByProjectId(CloudBuildClient.java:72)
 at cloudbuild.Test2.main(Test2.java:50)

Process finished with exit code 1
```

`The API does not exist or has not been published in the environment`表明该api未开放
