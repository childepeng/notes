# Spring Cloud Gateway

网关

## route

```
spring:
  cloud:
    gateway:
      routes:
        - id: user-service1     # 路由ID，唯一
          uri: http://localhost:8081/  # 目标地址
          predicates:                  # 断言，路由匹配条件
            - Path=/user/**            #
          filters:
            - Test=2
```

`-` 表示list

## predicates

## filter

### GatewayFilter  GlobalFilter



