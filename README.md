# HealthCheck Reborn

## ETA: March 1, 2022 / 释出时间：2022年5月1日

> Meets a brand new shortcut. / 遇见焕然一新的捷径。

Apple Shortcut implement for WTU Health Check.

武纺健康打卡的苹果快捷指令实现。

## Available On / 适用于

iOS/iPadOS 13 and later, macOS 12.0+

Tested on

- [x] iPhone XS - iOS 15.1
- [x] iPad (8th generation) - iPadOS 15.3.1
- [x] iPhone 6s - iOS 13.2.2
- [x] iPhone 12 Pro - iOS 14.3 [Simulator]
- [x] iPhone 13 Pro Max - iOS 15.0 [Simulator]
- [x] iPhone 8 - iOS 14.3 [Simulator]

## Privacy / 隐私

### Permissions 权限

- 🗂 Run another shortcut 运行其他快捷指令

  This shortcut will access to: / 此快捷指令会运行：

  ① Dependency 1 依赖1

  ② Dependency 2 依赖2

- 🌍 Network connection 网络连接

  This shortcut will access to: / 此快捷指令会访问：

  ① auth.wtu.edu.cn

  ② ehall.wtu.edu.cn

  ③ bhshare.cn

  **No other sites will be accessed**

  **不会访问除此以外的其他站点**

   - 📣 Notification 通知

### Password Security 密码安全

- Your password is **only sent to "auth.wtu.edu.cn"** for authorization, this shortcut **will not** send your password to any third party

  您的密码**只会发送给"auth.wtu.edu.cn"**用于信息系统认证，此快捷指令**不会**将您的密码发给任何第三方

- The password encryption method is powered by Dependency 1, written in JavaScript, **all done in the local**

  密码加密方法由依赖1提供，采用JavaScript编写，**全程在本地完成**

- Because of the restriction of Apple Shortcut, the password will be displayed in clear text in shortcut. Please protect your personal information.

  由于快捷指令的限制，密码在快捷指令中会以明文显示，请注意保护个人信息。

## Notice & Tips / 注意与建议

⚠️**By using this shortcut you agree to grant the permission above and allow it to save your password**

⚠️**使用此快捷指令即代表你同意授予以上权限并允许其保存你的密码**

- Project Structure 项目结构

  - **Main shortcut 主快捷指令**

    [下载](https://www.icloud.com/shortcuts/ab882a6a91e648cf8ca18b47835e8906)

  - **Dependency 1 依赖1**

    [下载](https://www.icloud.com/shortcuts/e50aec68be6d40c2b208929c30bee578)

  - **Dependency 2 依赖2** 

    [下载](https://www.icloud.com/shortcuts/1dff44e18bfd42c688134d7e3ba72216)

- Before you run the Main shortcut, please **make sure Dependency 1 & Dependency 2 are added to Library**

  在运行主快捷指令前请**确保依赖1和依赖2已经添加到快捷指令中**

- Change the device language to **Chinese** to run it correctly (SC,TC is fine)

  系统语言需为**中文**才能保证正常运行（繁体简体都可以）

- Configuration dialog will pop-up when you add the first time

  首次添加快捷指令时，会弹出配置对话框

  If you want to reconfigure, go to "Details">"Setup">"Customise Shortcut..." to pop it again

  如果你想重新配置，前往“详细信息”>“设置”>“自定快捷指令...”以再次弹出配置框

- Adding Main shortcut to **"Automation"** can make it run daily

  将主快捷指令加入**"自动化"**可使其每天运行

- More in comments...

  更多说明请见注释...

- Allow secondary development and distribution, need to indicate the source

  允许二次开发并分发，需要注明来源

## Screenshots / 运行截图

<img src="https://www.helloimg.com/images/2022/04/30/R2211b.png" width="40%" height="40%" /><img src="https://www.helloimg.com/images/2022/04/30/R22PkK.png" width="40%" height="40%" />
<img src="https://www.helloimg.com/images/2022/04/30/R2T45c.png" width="40%" height="40%" /><img src="https://www.helloimg.com/images/2022/04/30/R2T2iD.png" width="40%" height="40%" />
	

## 常见问题 (Chinese only)

- Q: 获取快捷指令时提示“此快捷指令无法打开，因为您的“快捷指令”安全性设置不允许不受信任的快捷指令”

  A: 前往“设置”>“快捷指令”>打开“允许不受信任的快捷指令”，如果开关是灰色的，你需要先运行一次快捷指令，具体教程可以百度。

- Q: 依赖2的token是什么，可以不配置吗？

  A: 因为快捷指令目前无法识别验证码，需要用到第三方接口(bhshare.cn)。而第三方接口每天是有限额的(100次/天)，所以需要申请token来使用。[点击此处申请token](http://www.bhshare.cn/imgcode/gettoken/)

  ​	如果觉得麻烦可以不申请token，在那一栏直接写free就好了，不过限额会少一些。

- Q: 运行时提示“找不到快捷指令”

  A: 首先查看自己的快捷指令库中是否有“依赖1”和“依赖2”，如果没有请下载导入进来。如果已有依赖1和依赖2或导入后还是提示，可以把主快捷指令、依赖1、依赖2都删掉，然后按 依赖1,依赖2,主快捷指令 的顺序添加。如果还是不行，可以去主快捷指令中手动指定快捷指令。

- Q: 提示“错误-用户名或密码错误”

  A: 首先检查用户名或密码是否输入错误，注意密码框不能有换行，如果有换行需要把它删掉。如果这个问题一直发生，请在发送issue。

- Q: 提示“错误-验证码接口异常”

  A: 检查“依赖2”的token是否填写正确。如果你用的是free，代表接口使用达到限额，可以去[此处](http://www.bhshare.cn/imgcode/gettoken/)申请token。

- Q: 提示“错误-验证码错误”

  A: 可以再试一次，如果问题仍然存在，可能是验证码接口方出现故障，请发送issue。

- Q: 提示“错误-系统语言不是中文”

  A: 将系统语言换成简体中文/繁體中文即可，如果问题仍然存在，可能是你选取的位置有误。

- Q: 提示“错误-提交打卡申请时发生错误”

  A: 可以再试一次，如果问题仍然存在，请发送issue（附上错误信息）。
