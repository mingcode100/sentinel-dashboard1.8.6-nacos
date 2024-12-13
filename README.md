# 前言

从 https://github.com/max-holo/sentinel-dashboard1.8.6-nacos fork而来，增加了docker支持

## 改造后效果

支持使用nacos作为动态数据源的规则设置，支持流控规则、熔断规则、热点规则、系统规则、授权规则、网关API分组规则、网关流控规则

## 注意事项
在使用SpringCloud Gateway和SpringCloud Sentinel时，官方不支持针对4xx,5xx响应码做异常统计熔断，可以自行实现，可参考[issue1842](https://github.com/alibaba/Sentinel/issues/1842) [issue2537](https://github.com/alibaba/Sentinel/issues/2537#issuecomment-1205963072)

# Sentinel 控制台

## 0. 概述

Sentinel 控制台是流量控制、熔断降级规则统一配置和管理的入口，它为用户提供了机器自发现、簇点链路自发现、监控、规则配置等功能。在 Sentinel 控制台上，我们可以配置规则并实时查看流量控制效果。

## 1. 编译和启动

### 1.1 如何编译

使用如下命令将代码打包成一个 fat jar:

```bash
mvn clean package
```

### 1.2 如何启动

使用如下命令启动编译后的控制台：

```bash
java -Dserver.port=8080 \
-Dcsp.sentinel.dashboard.server=localhost:8080 \
-Dproject.name=sentinel-dashboard \
-jar target/sentinel-dashboard.jar
```

上述命令中我们指定几个 JVM 参数，其中 `-Dserver.port=8080` 是 Spring Boot 的参数，
用于指定 Spring Boot 服务端启动端口为 `8080`。其余几个是 Sentinel 客户端的参数。

为便于演示，我们对控制台本身加入了流量控制功能，具体做法是引入 Sentinel 提供的 `CommonFilter` 这个 Servlet Filter。
上述 JVM 参数的含义是：

| 参数 | 作用 |
|--------|--------|
|`-Dcsp.sentinel.dashboard.server=localhost:8080`|向 Sentinel 接入端指定控制台的地址|
|`-Dproject.name=sentinel-dashboard`|向 Sentinel 指定应用名称，比如上面对应的应用名称就为 `sentinel-dashboard`|

全部的配置项可以参考 [启动配置项文档](https://github.com/alibaba/Sentinel/wiki/%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%A1%B9)。

经过上述配置，控制台启动后会自动向自己发送心跳。程序启动后浏览器访问 `localhost:8080` 即可访问 Sentinel 控制台。

从 Sentinel 1.6.0 开始，Sentinel 控制台支持简单的**登录**功能，默认用户名和密码都是 `sentinel`。用户可以通过如下参数进行配置：

- `-Dsentinel.dashboard.auth.username=sentinel` 用于指定控制台的登录用户名为 `sentinel`；
- `-Dsentinel.dashboard.auth.password=123456` 用于指定控制台的登录密码为 `123456`；如果省略这两个参数，默认用户和密码均为 `sentinel`；
- `-Dserver.servlet.session.timeout=7200` 用于指定 Spring Boot 服务端 session 的过期时间，如 `7200` 表示 7200 秒；`60m` 表示 60 分钟，默认为 30 分钟；

## 2. 客户端接入

选择合适的方式接入 Sentinel，然后在应用启动时加入 JVM 参数 `-Dcsp.sentinel.dashboard.server=consoleIp:port` 指定控制台地址和端口。
确保客户端有访问量，**Sentinel 会在客户端首次调用的时候进行初始化，开始向控制台发送心跳包**，将客户端纳入到控制台的管辖之下。

客户端接入的详细步骤请参考 [Wiki 文档](https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0#3-%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%8E%A5%E5%85%A5%E6%8E%A7%E5%88%B6%E5%8F%B0)。

## 3. 验证是否接入成功

客户端正确配置并启动后，会**在初次调用后**主动向控制台发送心跳包，汇报自己的存在；
控制台收到客户端心跳包之后，会在左侧导航栏中显示该客户端信息。如果控制台能够看到客户端的机器信息，则表明客户端接入成功了。

更多：[控制台功能介绍](./Sentinel_Dashboard_Feature.md)。

## sentinel-dashboard 部署
环境变量
```
TZ	Asia/Shanghai
SERVER_PORT	8858
NACOS_USERNAME	nacos
NACOS_SERVER_ADDR	xxxxxxxxxxx
NACOS_PASSWORD	xxxxxxxxx
NACOS_CONFIG_NAMESPACE	xxxxxxxxxx
AUTH_USERNAME	xxxxxxxxxxxx
AUTH_PASSWORD	xxxxxxxx
```

## 网关接入
配置文件
```yaml
sentinel:
  nacos:
    username: xxxxxxxxxxxxxxx
    password: xxxxxxxxxxxxxxxxx
    server-addr: xxxxxxxxxxx
    group: xxxxxxxxxx
    namespace: xxxxxxxxxxxx


spring:
  cloud:
    sentinel:
      eager: true
      transport:
        port: 8719
        dashboard: xxxxxxxxxxxxxxxxxxxxxxxxx:8858
      #        dashboard: 127.0.0.1:8858
      datasource:
        ds1:
          nacos:
            server-addr: ${sentinel.nacos.server-addr}
            namespace: ${sentinel.nacos.namespace}
            data-id: ${spring.application.name}-gateway-flow-rules
            group-Id: ${sentinel.nacos.group}
            rule-type: gw-flow
            data-type: json
            username: ${sentinel.nacos.username}
            password: ${sentinel.nacos.password}
        ds2:
          nacos:
            server-addr: ${sentinel.nacos.server-addr}
            namespace: ${sentinel.nacos.namespace}
            data-id: ${spring.application.name}-gateway-api-rules
            group-Id: ${sentinel.nacos.group}
            rule-type: gw-api-group
            data-type: json
            username: ${sentinel.nacos.username}
            password: ${sentinel.nacos.password}
        ds3:
          nacos:
            server-addr: ${sentinel.nacos.server-addr}
            namespace: ${sentinel.nacos.namespace}
            data-id: ${spring.application.name}-degrade-rules
            group-Id: ${sentinel.nacos.group}
            rule-type: degrade
            data-type: json
            username: ${sentinel.nacos.username}
            password: ${sentinel.nacos.password}
        ds4:
          nacos:
            server-addr: ${sentinel.nacos.server-addr}
            namespace: ${sentinel.nacos.namespace}
            data-id: ${spring.application.name}-system-rules
            group-Id: ${sentinel.nacos.group}
            rule-type: system
            data-type: json
            username: ${sentinel.nacos.username}
            password: ${sentinel.nacos.password}

feign:
  sentinel:
    enabled: true
```
```java

import com.alibaba.cloud.sentinel.SentinelProperties;
import com.alibaba.cloud.sentinel.datasource.config.DataSourcePropertiesConfiguration;
import com.alibaba.cloud.sentinel.datasource.config.NacosDataSourceProperties;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.ApiDefinition;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.GatewayApiDefinitionManager;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayFlowRule;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayRuleManager;
import com.alibaba.csp.sentinel.adapter.gateway.sc.SentinelGatewayFilter;
import com.alibaba.csp.sentinel.adapter.gateway.sc.exception.SentinelGatewayBlockExceptionHandler;
import com.alibaba.csp.sentinel.datasource.ReadableDataSource;
import com.alibaba.csp.sentinel.datasource.nacos.NacosDataSource;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import com.alibaba.nacos.api.PropertyKeyConst;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.web.reactive.result.view.ViewResolver;

import javax.annotation.PostConstruct;
import java.util.*;

@Slf4j
@Configuration
@EnableConfigurationProperties(SentinelProperties.class)
public class SentinelGatewayConfiguration {

    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;

    public SentinelGatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                        ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        // Register the block exception handler for Spring Cloud Gateway.
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }

    @Autowired
    private SentinelProperties nacosProperties;

    private final String groupId = "SENTINEL_GROUP";

    public static final String PARAM_FLOW_POSTFIX = "-param-rules";
    public static final String FLOW_DATA_ID_POSTFIX = "-flow-rules";
    public static final String DEGRADE_DATA_ID_POSTFIX = "-degrade-rules";
    public static final String SYSTEM_FULE_DATA_ID_POSTFIX = "-system-rules";
    public static final String AUTHORITY_DATA_ID_POSTFIX = "-authority-rules";
    public static final String GATEWAY_FLOW_DATA_ID_POSTFIX = "-gateway-flow-rules";
    public static final String GATEWAY_API_DATA_ID_POSTFIX = "-gateway-api-rules";
    public static final String DASHBOARD_POSTFIX = "-dashboard";

    public static final String PARAM_FLOW_DATA_ID_POSTFIX = "-param-rules";
    public static final String CLUSTER_MAP_DATA_ID_POSTFIX = "-cluster-map";


    @Value("${spring.application.name}")
    private String applicationName;

    @Value("${spring.application.name}"+ GATEWAY_FLOW_DATA_ID_POSTFIX)
    private String flowDataId;

    @Value("${spring.application.name}"+ GATEWAY_API_DATA_ID_POSTFIX)
    private String apiDataId;

    @Value("${spring.application.name}"+ PARAM_FLOW_POSTFIX)
    private String paramDataId;

    @Value("${spring.application.name}"+"-cluster-client-config")
    private String configDataId;

    private final String namespaceSetDataId = "cluster-server-namespace-set";
    private final String serverTransportDataId = "cluster-server-transport-config";


    @PostConstruct
    public void init(){
        NacosDataSourceProperties nacosDataSourceProperties = this.nacosProperties.getDatasource()
                .values()
                .stream()
                .filter(e -> e.getNacos() != null)
                .findFirst()
                .map(DataSourcePropertiesConfiguration::getNacos)
                .orElseThrow(() -> new IllegalStateException("缺少sentinel的NACOS配置"));

        Properties properties = new Properties();
        Optional.ofNullable(nacosDataSourceProperties.getEndpoint())
                .ifPresent(e->{properties.setProperty(PropertyKeyConst.ENDPOINT,e);});
        Optional.ofNullable(nacosDataSourceProperties.getNamespace())
                .ifPresent(e->{properties.put(PropertyKeyConst.NAMESPACE,e);});
        Optional.ofNullable(nacosDataSourceProperties.getUsername())
                .ifPresent(e->{properties.put(PropertyKeyConst.USERNAME,e);});
        Optional.ofNullable(nacosDataSourceProperties.getPassword())
                .ifPresent(e->{properties.put(PropertyKeyConst.PASSWORD,e);});
        Optional.ofNullable(nacosDataSourceProperties.getAccessKey())
                .ifPresent(e->{properties.put(PropertyKeyConst.ACCESS_KEY,e);});
        Optional.ofNullable(nacosDataSourceProperties.getSecretKey())
                .ifPresent(e->{properties.put(PropertyKeyConst.SECRET_KEY,e);});
        Optional.ofNullable(nacosDataSourceProperties.getServerAddr())
                .ifPresent(e->{properties.put(PropertyKeyConst.SERVER_ADDR,e);});
//        initUseClusterTokenServer(properties);
        initEmbed(properties);
        log.info("sentinel gateway 配置已加载");
    }


    public void initEmbed(Properties properties){
        // Register client dynamic rule data source.
        initDynamicRuleProperty(properties);
    }


    private void initDynamicRuleProperty(Properties properties) {
        ReadableDataSource<String, Set<GatewayFlowRule>> ruleSource = new NacosDataSource<>(properties, groupId,
                flowDataId, source -> JSON.parseObject(source, new TypeReference<Set<GatewayFlowRule>>() {}));

        ReadableDataSource<String, Set<ApiDefinition>> apiDefinitionSource = new NacosDataSource<>(properties, groupId,
                apiDataId, source -> JSON.parseObject(source, new TypeReference<Set<ApiDefinition>>() {}));

        GatewayRuleManager.register2Property(ruleSource.getProperty());
        GatewayApiDefinitionManager.register2Property(apiDefinitionSource.getProperty());
    }
}
```