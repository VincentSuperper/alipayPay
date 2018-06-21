# 支付宝支付(手机网页支付)

## 准备工作

1. 创建应用

    开发者使用支付宝账号[登录](https://auth.alipay.com/login/index.htm?goto=https%3A%2F%2Fopenhome.alipay.com%2Fplatform%2FmanageApp.htm)开放平台（需实名认证的支付宝账号），根据实际需求创建应用（如“支付应用”）

    *备注：创建应用时的应用状态为“开发中”，无法在线上正式调用接口*

2. 填写应用基础信息

    *应用基础信息在开发应用过程中可以无需审核随时完善*

    ![基础信息](http://img01.taobaocdn.com/top/i1/LB1g6SSSpXXXXXXXVXXXXXXXXXX)

3. 添加应用功能

    ![应用功能](http://img01.taobaocdn.com/top/i1/LB11ceTSpXXXXcsXFXXXXXXXXXX)

4. 配置应用环境

    | 字段名称 | 字段描述 |
    | :-----: | :----: |
    | 应用网关 | 用于接收支付宝异步通知，例如口碑开店中，需要配置此网关来接收[开发者门店被动通知](https://docs.open.alipay.com/205/105251/#s1) |
    | 授权回调地址 | 第三方授权或用户信息授权后回调地址。授权链接中配置的redirect_uri的值必须与此值保持一致。(如：https://www.alipay.com) 注：当填入该地址时，系统会自动进行安全检测，详情请参考[安全检测](https://docs.open.alipay.com/316/106274/) |
    | RSA(SHA256)密钥 | 开发者要保证接口中使用的私钥与此处的公钥匹配，否则无法调用接口。可参考密钥的[生成与配置](https://docs.open.alipay.com/291/105971)，且接口参数sign_type=RSA2 |
    | RSA(SHA1)密钥 | 同上，且接口参数sign_type=RSA |

    界面请参考：

    ![开发配置](http://img01.taobaocdn.com/top/i1/LB1GmQHPVXXXXcJapXXXXXXXXXX)

    *TIPS：必须填写“接口加密方式”（加密方式只需填写一个），才可以提交审核*

5. 生成RSA密钥

    支付宝提供一键生成工具便于开发者生成一对RSA密钥，可通过下方链接下载密钥生成工具：

    [WINDOWS](http://p.tb.cn/rmsportal_6680_secret_key_tools_RSA_win.zip)

    [MAC_OSX](http://p.tb.cn/rmsportal_6680_secret_key_tools_RSA_macosx.zip)

    下载该工具后，解压打开文件夹，运行“RSA签名验签工具.bat”（WINDOWS）或“RSA签名验签工具.command”（MAC_OSX）。
    
    界面示例：

    ![生成工具](https://gw.alipayobjects.com/zos/skylark/6dbc42cc-6b9b-4691-83f1-e7b875e1a602/2018/png/e6b725d0-8257-4a71-b7a0-f5479c9d43d0.png)

    详细步骤：

    1.根据开发语言选择密钥格式

    2.选择密钥长度，新建应用请务必使用2048位（目前已使用1024位密钥长度的应用仍然可以正常调用接口，详情请见[开放平台接口签名方式升级公告](https://open.alipay.com/platform/announcement.htm?id=2)）

    3.点击 “生成密钥”，会自动生成商户应用公钥和应用私钥

    4.点击“打开密钥文件路径”，即可找到生成的公私钥。如图：

    ![公私钥](https://gw.alipayobjects.com/os/skylark/public/files/c39581e6f75edf301b213610b8a2c088)

    *TIPS：除了使用支付宝提供的一键生成密钥工具外，也可以[使用OpenSSL工具命令生成密钥](https://docs.open.alipay.com/291/106130)*

6. 应用申请上线

    应用开发完成后，请开发者自行进行验收并完成安全性检查，验收检查完成后，可“提交审核”。应用上线成功后，状态变为已上线，该状态下的应用能够调用生产环境的接口

## 接口说明

  请求地址：https://openapi.alipay.com/gateway.do

  API文档地址：https://docs.open.alipay.com/203/107090/

## 开发步骤

1. 前端页面调用服务器支付接口

    先在前端页面把订单相关的信息收集好，调用服务器支付接口把所有支付所需的参数传到服务器，经过服务器处理后，返回一条url到前端用于调起支付宝收银台

2. 服务器处理订单数据生成支付url

    *注：这里使用的是[alipay-mobile](https://github.com/Luncher/alipay)这个第三方库*

    - 安装

          npm i alipay-mobile -d --save

    - 初始化

          const fs = require('fs')
          const Alipay = require('alipay-mobile')
          const read = filename => {
            return fs.readFileSync(path.resolve(__dirname, filename))
          }

          //app_id: 开放平台 appid
          //appPrivKeyFile: 你的应用私钥
          //alipayPubKeyFile: 蚂蚁金服公钥
          const options = {
            app_id: '2016080100137766',
            appPrivKeyFile: read('./keys/app_priv_key.pem'),
            alipayPubKeyFile: read('./keys/alipay_public_key.pem')
          }
          const service = new Alipay(options)

    - 创建网页订单 createWebOrderURL

      *该接口用于支付宝手机网页支付，服务端调用该接口生成一个URL返回给客户端, 客户端拿到该URL之后跳转到该URL发起支付请求。支付结束支付宝会跳转到客户端填写的return_url*

      示例：

          const data = {
            subject: '辣条',
            out_trade_no: '1232423',
            total_amount: '100'
          }
          const basicParams = {
            return_url: 'http://xxx.com'
          }
          return service.createWebOrderURL(data, basicParams)
          .then(result => {
            assert(result.code == 0, result.message)
            assert(result.message == 'success', result.message)
            return result
          })

      此处返回的result里会带有用于调起支付宝收银台的url，前端在收到这个url之后，跳转到该url即可：

          location.assign(url)

3. 支付宝异步通知

    对于手机网站支付产生的交易，支付宝会根据原始支付API中传入的异步通知地址notify_url，通过POST请求的形式将支付结果作为参数通知到商户系统

    - 异步通知参数说明

      https://docs.open.alipay.com/203/105286/

    - 异步通知页面特性

      - 必须保证服务器异步通知页面（notify_url）上无任何字符，如空格、HTML标签、开发系统自带抛出的异常提示信息等
      - 支付宝是用POST方式发送通知信息，因此该页面中获取参数的方式，如：request.Form(“out_trade_no”)、$_POST[‘out_trade_no’]
      - 支付宝主动发起通知，该方式才会被启用
      - 只有在支付宝的交易管理中存在该笔交易，且发生了交易状态的改变，支付宝才会通过该方式发起服务器通知（即时到账交易状态为“等待买家付款”的状态默认是不会发送通知的）
      - 服务器间的交互，不像页面跳转同步通知可以在页面上显示出来，这种交互方式是不可见的
      - 第一次交易状态改变（即时到账中此时交易状态是交易完成）时，不仅会返回同步处理结果，而且服务器异步通知页面也会收到支付宝发来的处理结果通知
      - 程序执行完后必须打印输出“success”（不包含引号）。如果商户反馈给支付宝的字符不是success这7个字符，支付宝服务器会不断重发通知，直到超过24小时22分钟。一般情况下，25小时以内完成8次通知（通知的间隔频率一般是：4m,10m,10m,1h,2h,6h,15h）
      - 程序执行完成后，该页面不能执行页面跳转。如果执行页面跳转，支付宝会收不到success字符，会被支付宝服务器判定为该页面程序运行出现异常，而重发处理结果通知
      - cookies、session等在此页面会失效，即无法获取这些数据
      - 该方式的调试与运行必须在服务器上，即互联网上能访问
      - 该方式的作用主要防止订单丢失，即页面跳转同步通知没有处理订单更新，它则去处理
      - 当商户收到服务器异步通知并打印出success时，服务器异步通知参数notify_id才会失效。也就是说在支付宝发送同一条异步通知时（包含商户并未成功打印出success导致支付宝重发数次通知），服务器异步通知参数notify_id是不变的。

4. 异步返回结果的验签

    *此处使用的还是[alipay-mobile](https://github.com/Luncher/alipay)这个第三方库*

    - 异步通知校验 makeNotifyResponse

      示例：

          const params = {
            sign: 'xxxxxxxx',
            sign_type: 'xxxxx',
            ...
          }

          return service.makeNotifyResponse(params)
    
    - 异步通知应答

      在接收到蚂蚁金服服务器的订单状态变更通知之后，需要进行应答，有两种(成功、失败)应答类型：

          import AlipayConfig from 'alipay-mobile/config'

          console.log(AlipayConfig.ALIPAY_NOTIFY_SUCCESS) // 'success'

          console.log(AlipayConfig.ALIPAY_NOTIFY_FAILURE) // 'failure'

