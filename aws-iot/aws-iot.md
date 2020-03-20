# AWS-IOT

## 设备注册

AWS IoT 设备预置涉及创建和注册以下实体：

* 证书。您可以使用现有证书预置设备，也可以让 AWS IoT 为您创建并注册一张证书。
* 附加到该证书的策略。
* 事物 \(设备\) 的唯一标识符。
* 事物的一组属性，包括现有事物类型和组。

要预配置设备，请创建一个描述您的设备所需的资源的模板。设备需要事物、证书以及一个或多个策略。_事物_ 是注册表中的一个条目，其中包含描述设备的属性。设备使用证书通过 AWS IoT 进行身份验证。策略确定设备在 AWS IoT 中可执行的操作。

模板包含在将模板用于预配置设备时替换的变量。字典 \(映射\) 用于为模板中使用的变量提供值。您可以使用同一个模板来预配置多个设备。只需在字典中为模板变量传递不同的值。AWS IoT 提供了三种方法来预置设备：

* 使用预置模板进行单一事物预置。 如果您只需**一次预置一台设备**，那么这是一个很好的选择。
* 使用在首次连接到 AWS IoT 时注册和预置设备的模板进行即时预置 \(JITP\)。 如果您**需要注册大量设备，但没有可以整合到批量预置列表中的有关这些设备的信息**，那么这是一个很好的选择。
* 批量预置。[利用 AWS IoT Device Management 服务轻松部署设备组 \| AWS 上的物联网](https://aws.amazon.com/cn/blogs/china/deploy-fleets-easily-with-aws-iot-device-management-services/) 此选项可让您指定存储在 S3 存储桶内的文件中的单一事物预置模板值。**如果您有大量已知设备，**并且可以将它们所需的特性整合到列表中，则此方法很有用。

如果您需要预置大量设备，那么即时预置和批量预置是更好的选择。AWS IoT 还提供了一个 [RegisterThing](https://docs.aws.amazon.com/iot/latest/apireference/API_RegisterThing.html) API，可用于以编程方式预置单个设备。

### JITR（即时注册）

1. 何时使用设备证书的即时注册？ **当用户希望使用从第三方机构购买或者自签发的 CA 证书，并将由该 CA 证书签发的设备证书的设备连接到 AWS IoT Core 时，可以利用即时注册功能来实现。** 如果希望直接利用 AWS IoT CA 证书签发的设备证书对设备进行注册激活，可以参考 [Certificate Vending Machine 方案](https://aws.amazon.com/cn/blogs/china/certification-vending-machine-intelligent-device-access-aws-iot-platform-solution/)。
2. 实现步骤：
   1. 创建 CA 证书并在 AWS IoT Core 上注册和激活。
   2. 使用该 CA 证书签发设备证书并安装在 IoT 设备上。
   3. 创建 Lambda 函数实现设备证书在 AWS IoT Core 上的自动注册。
   4. IoT 设备与 AWS IoT Core 的第一次连接。

### JITP（即时部署）

1. 何时使用设备证书的即时注册？ 利用即时注册（JITR）功能，可以快速的进行设备证书注册及设备上线。但是配置相关 Lambda 函数的方式相对复杂。使用本文介绍的即时部署（JITP）功能，可以简化 IoT 规则和 Lambda 函数的步骤，**直接在注册设备 CA 证书的同时附加上一个自定义好的模版。在设备第一次连接 AWS IoT 平台的时候，JITP 会参照该模版的定义来完成设备证书的注册和设备在云端的创建过程。**
2. JITP 的实现步骤如下：
   1. 创建 CA 证书、配置模版和相应的 IAM 权限。
   2. 在 AWS IoT 平台上注册和激活 CA 证书并附加模版。
   3. 使用该 CA 证书签发设备证书并安装在 IoT 设备上。
   4. IoT 设备与 AWS IoT 平台的第一次连接。

### Certificate Vending Machine

1. 何时应当使用 Certificate Vending Machine？

   **对于部分已经出厂的 IoT 设备，可能在生产过程中没有预装任何证书，但是又希望这些设备连接至 AWS IoT 平台。** 此时，Certificate Vending Machine \(简称 CVM\) 可以作为给 IoT 设备写入相关证书的可行方案，让 IoT 设备自行向 CVM 服务器申请 AWS IoT 平台 CA 根证书与 IoT 设备证书，并且通过 AWS IoT 管理平台控制设备证书权限，确保物联网通信安全。

但是需要注意，由于**默认情况下，原设备没有任何证书进行申请证书阶段的 TLS 认证，所以使用 CVM 的过程中需要注意三点**：

1. IoT 设备与 CVM 系统通信时，原生并没有安全保护手段，所以需要在受信的 DNS 环境下进行，以防中间人攻击。或者采用其他安全链接的方式，例如使用 HTTPS 与 CVM 服务器交互（需要额外证书）。
2. IoT 设备在利用 CVM 系统申请证书时， IoT 设备本身应该具备唯一标识符用于设备的身份标识，例如序列号，client ID 或者 product ID 等，通过该身份标识符进行证书申请及策略绑定。
3. 所有通过 CVM 系统申请获发的 IoT 设备证书的 CA 根证书，只可以为 AWS IoT 平台默认使用的 CA 根证书（VeriSign Class 3 Public Primary G5 证书）。
4. 如果需要使用自定义的 CA 根证书来进行 IoT 设备证书的签发，请参考另一篇文档 – JITR 证书注册方式，即，为每个设备在出厂前写入独立的 IoT 设备证书和 CA 根证书。

**为了使 CVM 服务端更具稳定与扩展性，可以使用 AWS API Gateway 和 Lambda 来部署 CVM。通过这种方式，不需要长时间维护和管理部署在 EC2 上的 CVM，而是通过 IoT 设备的证书申请的需求，灵活的调配 AWS 上的服务资源。**流程：

1. IoT 设备发送相应 API 请求到 API Gateway 申请 IoT 证书
2. AWS API Gateway 调用申请证书的 Lambda 向 IoT 平台发起证书申请
3. Lambda 接收到请求后， 查询 DynamoDB 校验请求合法性
4. 确认当前请求合法之后，通过 API 的形式，向 IoT 平台申请证书
5. IoT 平台返回新创建的 IoT 设备证书，以及 IoT 设备证书对应的 Certificate ID
6. 通过查找 DynamoDB 中预先创建的对应关系，根据产品序列号，为当前申请到的证书附加对应的 Thing Name（设备属性）以及 Policy（权限）
7. 利用 Lambda 进行 IoT 设备证书的策略的绑定以及 DynamoDB 关联关系表里的证书状态标识符更新
8. 最终 CVM 将 IoT 平台 CA 根证书和 IoT 设备证书返回给 IoT 设备

**使用 EC2 替代 API Gateway 与 Lambda 的解决方案，其工作流程与搭建 lambda 的模式基本一致，仅在 IoT 设备与 CVM 系统通信时的调用关系上有所区别。**流程：

1. IoT 设备向 CVM 服务器申请 IoT 设备证书
2. EC2 接收到请求后，访问 MySQL 校验请求合法性
3. 确认当前请求合法之后，CVM 通过 API 的形式，向 IoT 平台发起获取 IoT 设备证书的请求
4. IoT 平台返回当前 IoT 设备对应的证书，以及当前证书的 Certificate ID
5. 通过查找 MySQL 中预先创建的对应关系，根据产品序列号，为当前证书附加对应的 Thing Name（产品属性） 以及 Policy（权限）
6. CVM 更新 MySQL 的关联关系表中的当前设备的所有关联信息以及证书状态标识符
7. 最终 CVM 将 IoT CA 根证书和设备证书返回给 IoT 设备

## JITP 搭建步骤（即时部署）

* [配置 AWS CLI](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-configure.html)
* [即时预配置](https://docs.aws.amazon.com/zh_cn/iot/latest/developerguide/jit-provisioning.html)
* [使用 AWS IoT Core 即时预配置](https://aws.amazon.com/cn/blogs/china/setting-up-just-in-time-provisioning-with-aws-iot-core/)
* [预配置模板](https://docs.aws.amazon.com/zh_cn/iot/latest/developerguide/provision-template.html)
* [使用模版自动化 IoT 设备创建及证书注册过程](https://aws.amazon.com/cn/blogs/china/aws-iot-series-2/)

1. 创建 CA 证书、配置模版和相应的 IAM 权限。
2. 在 AWS IoT 平台上注册和激活 CA 证书并附加模版。
3. 使用该 CA 证书签发设备证书并安装在 IoT 设备上。
4. IoT 设备与 AWS IoT 平台的第一次连接。

**由于 JITP 是通过模版定义来注册设备证书和创建设备，因此模版中相应的权限必不可少。**

**在准备好角色和权限后我们开始创建模版，这个模版会在之后注册 CA 证书的时候一起提交给 AWS IoT Core。**

**通过命令导入 CA 证书和中间证书完成 CA 证书的注册和激活。同时，通过设置–allow-auto-registration 的方式，开启设备连接至 IoT Core 时设备证书的自动注册，并通过–registration-config 绑定模版到 CA 证书上。**

预配置模板是一个 JSON 文档，该文档使用参数来描述设备与 AWS IoT 交互时必须使用的资源。模板包含两个部分：Parameters 和 Resources。例如：我们在使用两个预置参数 AWS::IoT::Certificate::Country 和 AWS::IoT::Certificate::Id，而且我们将在“资源”部分中使用它们。JITP 工作流会将参考替换为从证书中提取的值，并且会预置模板中指定的资源。

更具体地说，JITP 工作流将创建：

* 注册一个证书并将其状态设置为 PENDING\_ACTIVE。
* 一个事物资源。
* 一个策略资源。

然后，它将会：

* 将策略附加到证书中。
* 将证书附加到事物中。
* 将证书状态更新为 ACTIVE。

现在，我们将整个模板与我们从前面的步骤中获得的角色 ARN 一起放入本地文件 provisioning-template.json 中。

示例 1：

```text
{
"templateBody":"{ \"Parameters\" : { \"AWS::IoT::Certificate::Country\" : { \"Type\" : \"String\" }, \"AWS::IoT::Certificate::Id\" : { \"Type\" : \"String\" } }, \"Resources\" : { \"thing\" : { \"Type\" : \"AWS::IoT::Thing\", \"Properties\" : { \"ThingName\" : \"MeshThing\", \"AttributePayload\" : { \"version\" : \"v1\", \"device type\" : \"mesh\", \"country\" : {\"Ref\" : \"AWS::IoT::Certificate::Country\"}} } }, \"certificate\" : { \"Type\" : \"AWS::IoT::Certificate\", \"Properties\" : { \"CertificateId\": {\"Ref\" : \"AWS::IoT::Certificate::Id\"}, \"Status\" : \"ACTIVE\" } }, \"policy\" : {\"Type\" : \"AWS::IoT::Policy\", \"Properties\" : { \"PolicyName\" : \"MeshThingPolicy\" } } } }",
"roleArn":"arn:aws:iam::443455854268:role/MeshThingJITPRole"
}
```

示例 2：

```text
{
"templateBody":"{ \"Parameters\" : { \"AWS::IoT::Certificate::Country\" : { \"Type\" : \"String\" }, \"AWS::IoT::Certificate::Id\" : { \"Type\" : \"String\" } }, \"Resources\" : { \"thing\" : { \"Type\" : \"AWS::IoT::Thing\", \"Properties\" : { \"ThingName\" : {\"Ref\" : \"AWS::IoT::Certificate::Id\"}, \"AttributePayload\" : { \"version\" : \"v1\", \"country\" : {\"Ref\" : \"AWS::IoT::Certificate::Country\"}} } }, \"certificate\" : { \"Type\" : \"AWS::IoT::Certificate\", \"Properties\" : { \"CertificateId\": {\"Ref\" : \"AWS::IoT::Certificate::Id\"}, \"Status\" : \"ACTIVE\" } }, \"policy\" : {\"Type\" : \"AWS::IoT::Policy\", \"Properties\" : { \"PolicyDocument\" : \"{\\\"Version\\\": \\\"2012-10-17\\\",\\\"Statement\\\": [{\\\"Effect\\\":\\\"Allow\\\",\\\"Action\\\": [\\\"iot:Connect\\\",\\\"iot:Publish\\\"],\\\"Resource\\\" : [\\\"*\\\"]}]}\" } } } }",
"roleArn":"arn:aws:iam::123456789012:role/JITPRole"
}
```

示例 3：

```text
{
    "Parameters": {
        "AWS::IoT::Certificate::Country": {
            "Type": "String"
        },
        "AWS::IoT::Certificate::Id": {
            "Type": "String"
        }
    },
    "Resources": {
        "thing": {
            "Type": "AWS::IoT::Thing",
            "Properties": {
                "ThingName": "MeshThing",
                "AttributePayload": {
                    "version": "v1",
                    "device_type": "mesh",
                    "country": {
                        "Ref": "AWS::IoT::Certificate::Country"
                    }
                }
            }
        },
        "certificate": {
            "Type": "AWS::IoT::Certificate",
            "Properties": {
                "CertificateId": {
                    "Ref": "AWS::IoT::Certificate::Id"
                },
                "Status": "ACTIVE"
            }
        },
        "policy": {
            "Type": "AWS::IoT::Policy",
            "Properties": {
                "PolicyName": "MeshThingPolicy"
            }
        }
    }
}
```

注意事项：

1. 创建模板时，需要替换 `roleArn`
2. 调用 register-ca-certificate CLI 命令来注册 CA 证书时，会由于权限不足导致失败 `An error occurred (AccessDeniedException) when calling the RegisterCACertificate operation: User: arn:aws:iam::443455854268:user/InfiniteYuan is not authorized to perform: iam:PassRole on resource: arn:aws:iam::443455854268:role/MeshThingJITPRole`
3. 根证书 root.cert 是 AWS IoT 根证书。要下载该证书，单击[此处](https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) 或者 [AmazonRootCA1.pem](https://www.amazontrust.com/repository/AmazonRootCA1.pem)
4. `Shadow Register Subdev Delta Error: -12` 策略中没有影子相关权限
5. `endpoint` 应设置为 `aws iot describe-endpoint`

### 使用 CA 证书签发设备证书：

1. 创建一个设备证书的私钥 deviceCert.key 和对应的证书请求文件 deviceCert.csr：`openssl genrsa -out deviceCert.key 2048`
2. `openssl req -new -key deviceCert.key -out deviceCert.csr`
3. 使用 CA 证书，CA 证书私钥和证书请求文件签发设备证书 deviceCert.crt:`openssl x509 -req -in deviceCert.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out deviceCert.crt -days 365 -sha256`
4. **创建一个包含设备证书及其注册 CA 证书的文件，合并 CA 证书和设备证书到一个新的证书形成有效的证书链**: `cat deviceCert.crt rootCA.pem > deviceCertAndCACert.crt`
5. MQTT Mosquitto 客户端使用设备证书连接并发布到 AWS IoT Core 中：验证两次以上： `mosquitto_pub --cafile root.cert --cert deviceCertAndCACert.crt --key deviceCert.key -h <prefix>.iot.us-east-1.amazonaws.com -p 8883 -q 1 -t foo/bar -I anyclientID --tls-version tlsv1.2 -m "Hello" -d`
6. 连接并发布至 AWS IoT Core 后，预置工作流将在 TLS 握手期间自动预置模板中指定的资源。在示例中：

* 我们为设备创建了事物资源。
* 创建了示例 CA 证书签署的证书，并将该证书的状态设置为 ACTIVE。
* 创建了策略资源并将其附加到证书中，并且将证书附加到事物资源上。

