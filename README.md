[TOC]

# **1 准入规范**

## **1.1 Oauth2准入规范**

oauth2-enhance使用授权码授权，客户端凭据授权， 隐式授权或密码授权四种模式进行身份验证，分别如下：

> * 授权码授权模式：根据client_id获取授权码，通过授权code及client_id、client_secret获取token对象及用户附加信息
> * 客户端授权模式：根据client_id、client_secret获取token对象及用户附加信息
> * 隐式授权模式：根据client_id获取token对象及用户附加信息
> * 密码授权模式：根据client_id、client_secret、username、password获取token对象及用户附加信息

API网关须对接权限业务系统共同完成客户端用户身份授权和权限校验，根据授权模式的不同返回客户结果不同

1、授权码授权、客户端授权、隐式授权模式

> * 权限中心登记客户端应用相关信息，包括client_id、client_secret等
> * 网关通过client_id等调用权限中心接口，如若权限中心校验client_id成功并关联到相关用户附加信息返回；如若权限中心识别检验client_id失败，无任何返回

2、密码授权模式

> * 权限中心注册用户账号信息，包括user_id、username、password等
> * 网关通过username、password等调用权限中心接口，如若权限中心校验账号成功，再通过user_id等参数调用接口关联到相关用户附加信息并返回；如若权限中心识别检验用户账号失败，无任何返回

API网关与权限业务系统统一rest规范接口对接，发送/响应数据统一采用application/json;charset=UTF-8编码，具体接口规范如下：


### **1.1.1 用户验证**

#### **1.1.1.1 功能说明**

客户端发起用户请求信息(包括用户名、密码、验证码等)，经网关转发到应用中心校验用户合法性

#### **1.1.1.2 API请求URLS**

POST：/context-path/user_validation

备注：/context-path表示应用中心微服务上下文路径，未设置则为空

#### **1.1.1.3 API请求参数**

| 名称         | 位置 | 类型   | 描述         | 必填 | 取值范围 | 版本 |
| ------------ | ---- | ------ | ------------ | ---- | -------- | ---- |
| username     | body | string | 校验用户名   | Y    |          | 1.0  |
| password     | body | string | 校验用户密码 | Y    |          | 1.0  |
| verification | body | string | 验证码       | N    |          | 1.0  |

 

#### **1.1.1.4 API请求示例**
```json
{ 

  "username": "gancy",  

  "password": "tgb.258", 

  "verification": "abfd"

}
```


#### **1.1.1.5 API返回**

| 名称          | 位置 | 类型   | 描述             | 必填 | 取值范围   | 版本 |
| ------------- | ---- | ------ | ---------------- | ---- | ---------- | ---- |
| isAuthSuccess | body | string | 用户校验是否成功 | Y    | true/false | 1.0  |
| userId        | body | number | 用户ID           | N    |            | 1.0  |

 

#### **1.1.1.6 API返回示例**
```json
{

  "isAuthSuccess": true,

  "userId": 123423

}
```


### **1.1.2 获取用户附件信息**

#### **1.1.2.1 功能说明**

校验通过后，从应用中心微服务抓取相关用户附件信息，与token信息一并返回客户端

备注：该接口请求/响应body支持类型皆为application/json;charset=UTF-8

#### **1.1.2.2 API请求URLS**

POST：/context-path/user_append_info

备注：/context-path表示应用中心微服务上下文路径，未设置则为空

#### **1.1.2.3 API请求参数**

| 名称          | 位置 | 类型   | 描述                      | 必填 | 取值范围 | 版本 |
| ------------- | ---- | ------ | ------------------------- | ---- | -------- | ---- |
| client_id     | body | string | 客户端ID                  | Y    |          | 1.0  |
| client_secret | body | string | 客户端密码 为空则无此属性 | N    |          | 1.0  |
| user_id       | body | number | 用户ID 为空则无此属性     | N    |          | 1.0  |

 

#### **1.1.2.4 API请求示例**
```json
{ 

  "client_id": "SampleClientId",  

  "client_secret": "tgb.258", 

  "user_id": 123423

}
```


#### **1.1.2.5 API返回**

| 名称             | 位置             | 类型   | 描述             | 必填 | 取值范围 | 版本 |
| ---------------- | ---------------- | ------ | ---------------- | ---- | -------- | ---- |
| user_attach_info | body             | object | 用户附件信息     | Y    |          | 1.0  |
| ......           | user_attach_info | ...... | 用户附件信息body | Y    |          | 1.0  |

 

#### **1.1.2.6 API返回示例**
```json
{

   "user_attach_info": {......}

}
```


## **1.2 自动注册准入规范**

API网关支持3种自动注册机制

### **1.2.1 路由自动注册**

业务微服务按照规范依赖API网关starter包并启动，网关starter采集微服务所有controller内接口，实现批量自动添加服务路由。业务微服务pom依赖示例如下：

![网关starter依赖示例](./网关starter依赖.png)



### **1.2.2 服务自动注册**

业务微服务按照接入规范元数据配置，实现从微服务云注册中心拉取服务列表并自动添加注册，包括负载、单节点目标、服务的添加。目前支持3中微服务框架准入

1、原生云eureka微服务框架接入，单节点微服务yml配置示例如下：
![原生云单机配置示例](./原生云单机配置.png)

2、阿里云nacos微服务框架接入，单节点微服务yml配置示例如下：
![阿里云单机配置示例](./阿里云单机配置.png)

3、腾讯云consul微服务框架接入，单节点微服务yml配置示例如下：

![腾讯云单机配置示例](./腾讯云单机配置.png)




### **1.2.3 负载自动注册**

业务微服务按照接入规范元数据配置，实现从微服务云注册中心拉取服务节点集群列表并自动注册，包括负载、多节点(集群)目标、服务的添加。目前支持3种微服务框架准入

1、原生云eureka微服务框架接入，同一集群各服务节点yml或properties配置中元数据metadata-map下集群名称clusterName配置值相同，yml配置示例如下：

![原生云集群配置示例](./原生云集群配置.png)

2、阿里云nacos微服务框架接入，同一集群各服务节点yml或properties配置中元数据metadata下集群名称clusterName配置值相同，yml配置示例如下：

![阿里云集群配置示例](./阿里云集群配置.png)

3、腾讯云consul微服务框架接入，同一集群各服务节点yml或properties配置中标签值相同，yml配置示例如下：

![腾讯云集群配置示例](./腾讯云集群配置.png)
