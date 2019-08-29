---
title: Google Ad Manager API V4.4.0报错
tags:
 - problems
 - Ad platform
 - Google Ad Manager
urlname: solve-GAM-api-error
date: 2019-03-18 17:45:19
---

> 今天将Google Ad Manager（文中简称GAM） API 升级到了[V4.4.0](<https://github.com/googleads/googleads-java-lib/releases/tag/4.4.0>)，于是写个main准备验证下API，但是发现程序运行不起来，后来看了下[issue](https://github.com/googleads/googleads-java-lib/issues/169)中有个和我一样的问题，说是maven依赖的版本问题。本文主要记录下解决的过程

## 背景

项目采用的是`Spring Boot 2` /`Spring Cloud` ，`swagger2`作为API 文档管理工具(问题就是源于这个...)

引入GAM API的相关依赖：

```xml
<!--ad manager API-->
<dependency>
    <groupId>com.google.api-ads</groupId>
    <artifactId>ads-lib</artifactId>
    <version>4.4.0</version>
</dependency>
<dependency>
    <groupId>com.google.api-ads</groupId>
    <artifactId>dfp-axis</artifactId>
    <version>4.4.0</version>
</dependency>
```

## 问题

写个main运行下（打印GAM中所有的Key值）：

```java
public static void main(String[] args) {
        AdManagerServices adManagerServices = new AdManagerServices();
        AdManagerSession session = null;
        try {
            Credential oAuth2Credential = new OfflineCredentials.Builder()
                    .forApi(OfflineCredentials.Api.AD_MANAGER)
                    .fromFile()
                    .build()
                    .generateCredential();
            session = new AdManagerSession.Builder()
                    .fromFile()
                    .withOAuth2Credential(oAuth2Credential)
                    .build();
        } catch (OAuthException e) {
            e.printStackTrace();
            logger.error(String.format("Failed to create OAuth credentials. Check OAuth settings in the %s file. ",DEFAULT_CONFIGURATION_FILENAME),e);
        } catch (ValidationException e) {
            e.printStackTrace();
            logger.error(String.format("Invalid configuration in the %s file. ",DEFAULT_CONFIGURATION_FILENAME), e);
        } catch (ConfigurationLoadException e) {
            e.printStackTrace();
            logger.error(String.format("Failed to load configuration from the %s file. ",DEFAULT_CONFIGURATION_FILENAME), e);
        }


        CustomTargetingServiceInterface customTargetingService = adManagerServices.get(session,CustomTargetingServiceInterface.class);
        StatementBuilder statementBuilder = new StatementBuilder()
                .orderBy("id ASC")
                .limit(StatementBuilder.SUGGESTED_PAGE_LIMIT);
        CustomTargetingKeyPage keys = null;
        try {
            keys = customTargetingService.getCustomTargetingKeysByStatement(statementBuilder.toStatement());
            for (CustomTargetingKey customTargetingKey : keys.getResults()) {
                System.out.println("Key id <"+customTargetingKey.getId()
                        +"> Key name <"+customTargetingKey.getName()
                        +"> Key displayName <"+customTargetingKey.getDisplayName()
                        +"> Key status <"+customTargetingKey.getStatus()
                        +"> Key type <"+customTargetingKey.getType());
            }
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
```

错误信息(错误信息中的行数已修改成上面代码的位置)：

```
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkNotNull(Ljava/lang/Object;Ljava/lang/String;Ljava/lang/Object;)Ljava/lang/Object;
	at com.google.api.ads.common.lib.auth.OfflineCredentials$Api.<init>(OfflineCredentials.java:89)
	at com.google.api.ads.common.lib.auth.OfflineCredentials$Api.<clinit>(OfflineCredentials.java:81)
	at com.xxxxx.ads.api.weather.service.impl.KVService.main(KVService.java:5)

```

## 定位与解决

Google 了下错误信息，最后在[官方Java lib](https://github.com/googleads/googleads-java-lib)中有个和我相似的[issue](<https://github.com/googleads/googleads-java-lib/issues/169>).（花了好一会时间才找到这个，因此在以后查找问题的时候可以优先考虑先去相应的`Github`中的`issue`中去查找，这样更能快速找到解决办法），GAM API中使用了`guava`这个lib，最新的V4.4.0中使用的是`guava` 需要20.0以上的版本。按照里面给出的方法，使用`mvn dependency:tree`列出项目的依赖信息，然后搜索`guava`相关的，发现`swagger2` 2.7依赖的是18.0的`guava`:

![guava](https://ws2.sinaimg.cn/large/006tKfTcgy1g1772m4o4gj31fi0fvawh.jpg)

因此有两个解决办法(已更新到issue的[comment](https://github.com/googleads/googleads-java-lib/issues/169#issuecomment-473825603)中，没错 tryandcatch 就是我:joy:)：

- 将`swagger2`的版本升级，改用2.8.x及以上的版本(推荐)：

  ```xml
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.8.0</version>
  </dependency>
  ```

  

- 将`swagger2`中的`guava`依赖“排除”掉，使用指定的`guava`：

  ```xml
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.7.0</version>
      <exclusions>
          <exclusion>
              <groupId>com.google.guava</groupId>
              <artifactId>guava</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <!--specific guava version-->
  <dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>22.0</version>
  </dependency>
  ```

  



## 总结

- 使用第三方的API，查找问题的解决办法优先去issue或者官方论坛中寻找

- 列出项目的依赖关系：

  - 命令行：`mvn dependency:tree`
  - 网页查看:[mvnrepository](<https://mvnrepository.com/artifact/io.springfox/springfox-swagger2/2.9.1>)的`Compile Dependencies`部分

- 排除依赖：

  ```xml
  <dependency>
      <groupId>xx.xxx</groupId>
      <artifactId>xxx</artifactId>
      <version>version</version>
      <exclusions>
          <exclusion>
              <groupId>xx.xx.xx</groupId>
              <artifactId>xx</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

---