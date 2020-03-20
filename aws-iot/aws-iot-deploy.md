# AWS-IOT-deploy

## 概述

架构和业务逻辑的设计，整个方案会使用到如下 AWS 的相关服务，建议读者先了解其相关功能，便于理解整个方案。

* [IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html): 用于设备连接，设备管理，设备认证，消息转发。
* [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html): 提供设备配置信息，调用 AWS IoT Core API, DynamoDB API。
* [API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html): 管理维护 Restful API，并且触发 Lambda。
* [DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html): 存放用户和设备的绑定关系。
* [STS](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html): 获得 AWS 临时授权，用于设备连接至 IoT Core。
* [CloudFormation](https://aws.amazon.com/cloudformation/): 帮助用户在不同地区快速部署相同的架构、服务。

当用户首次拿到一台设备的时候，需要对设备进行初始化配网设置，在配网过程中，我们使用 AWS IoT API Programmatic Provisioning 自定义证书的激活流程。结合在配网过程中，用户的实际操作流程，每一步的具体技术实现细节如下:

1. 将设备变成 WIFI 热点模式，设备设置为静态 IP 地址，并监听配置命令
2. 手机连接至设备 WIFI（非家庭 WIFI）
3. 进入 APP, 输入家庭 WIFI 的 SSID, password，并选择地区信息（标注1）
4. APP 将配置信息发送到设备的静态 IP，设备返回 MAC 地址
5. 设备退出 WIFI 热点模式，连接至家庭 WIFI, 并且进入初始化状态
6. 设备从服务配置 Endpoint 获得该地区的配置信息，并存储到本地。此时需要将地区信息作为参数传给服务配置 Endpoint. 服务配置 Endpoint 全球只需部署一份（标注2） 上图中的 country 和 state 组成了服务端所需的地区信息。服务端收到地区信息后返回该地区对应的配置信息。配置信息中至少包含两个地址，一个是证书激活的地址，另外一个是 AWS IoT 的连接地址。
7. 设备向云端发起设备注册请求，云端调用 RegisterThing API 注册设备，并返回注册结果。\(标注3\)
8. 设备在获得成功注册的返回值后，通过 MQTT 连接至 AWS IoT（标注4）
9. APP 向云端查询设备是否注册成功，如果已经注册成功，则在数据库中建立用户和设备的绑定关系。（标注5）
10. APP 向云端发起请求，申请接入 AWS IoT 的临时授权 （标注6），Lambda 先查询用户和设备的绑定关系，然后再向 STS 申请临时授权。
11. APP 获得临时授权之后，连接到 AWS IoT Device Gateway, 初始化完成（标注7）

## 用户池与设备绑定

[认证授权专题\(三\) : Cognito Identity Pool + IoT Core 实现 Mobile 端用户对设备权限的精细化控制](https://aws.amazon.com/cn/blogs/china/cognito-identity-pool-iot-core-realize-mobile-user-control/)

[Configuring Cognito User Pools to Communicate with AWS IoT Core](https://aws.amazon.com/cn/blogs/iot/configuring-cognito-user-pools-to-communicate-with-aws-iot-core/)

通过cognito user pool，无需自己coding，即可轻松实现用户的注册、登录、注销等基本操作。Cognito Identity Pool可以与cognito user pool或是其他第三方账号\(如google，facebook\)做对接，利用IAM Role实现对AWS资源的精细化控制。本文同时使用cognito User Pool和cognito identity Pool，实现对Iot Core的访问管理。终端用户通过cognito user pool的用户池，获得登录token，通过此登录成功的token，拿到cognito Identity Pool Authorized Role的身份，使得他有权访问Iot Core并且只能发布消息到自己的设备控制topic。用户的登录ID和设备之间的绑定关系存储在AWS NoSQL数据库DynamoDB当中，用户只能发布消息到自己的Iot设备。

## 注意事项

使用链接中的 Web 示例，需要注意修改相关的变量，特别注意：

1. logout\_uri=[http://localhost:8000](http://localhost:8000/) 和 RedirectUriSignIn、RedirectUriSignOut 的值一定要与 用户池中的应用程序客户端中的回调 URL 相同（URL 末尾的 ‘/’ 特别注意）
2. TableName 一定要和 AWS DynamoDB 中创建的相同

