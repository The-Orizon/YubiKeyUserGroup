# 关于 Yubikey 4 的一些玄学

大概在两个月前自行入了一枚 [Yubikey 4][yk]，本来想着就是当 [OpenPGP Card][pgpc] 和 [U2F][u2f] 用的，结果发现 [Yubikey 4][yk] 的各种功能和用法简直是魔幻般的玄学。网上的一些长者的人生经验，包括官方钦定的文档，好像都没怎么系统地讲清楚 [Yubikey 4][yk] 的功能，这里我想把我两个月来折腾的心得分享一下。

## 功能

Yubikey 4 有很多功能，它们的关系如下：

```
Yubikey
  |- HID
  |   |- U2F
  |   |- OTP (2 Slots)
  |       |- Yubico OTP
  |       |- OATH HOTP
  |       |- Static Password
  |       |- Challenge-Response
  |           |- Yubico OTP
  |           |- HMAC-SHA1
  |
  |- Smart Card
      |- PIV (18 Slots, mainly 4 slots)
      |- OpenPGP Card (3 Slots: E, S, A)
```


### U2F

Universal 2FA 是 [FIDO Alliance][fido] 制定的新一代基于 Challenge-Response 的二步验证机制，[Yubico][yubico] 作为 FIDO 的成员之一当然是不遗余力地推出了多款 U2F USB Token，[Yubikey 4][yk] 就是一款集成了 [U2F][u2f] 的产品。[Yubikey 4][yk] 的 U2F 功能是开箱即用的；插入 Yubikey，打开网站的设备注册，当绿灯闪烁的时候轻触按钮，便可提高账户安全性。

### OTP

[Yubikey 4][yk] 的 OTP 功能并不仅仅指其 [Yubico OTP][yubico-otp] 功能，它包括：

- [Yubico OTP][yubico-otp]
- [OATH HOTP][oath-hotp]
- [静态密码][static-pass]
- [Challenge-Response][chalresp]

在 OTP 功能中，共有 **2 个 Slot**，可以设定以上任意功能到两个 Slot 中。

#### Yubico OTP

~~“最近出现一种人，贴出一串不明字符串来炫耀自己拥有某种硬件。”~~

这就是 [Yubico OTP][yubico-otp]，它以 [YubiCloud][ycloud] 作为中心验证由 Token 传送到服务器的一次性密码。这一功能是默认在 [Yubikey 4][yk] 内的 Slot 1 开启的，所以购买 Yubikey 后插上电脑，第一件事就是轻触按钮发送一串不明字符串来炫耀啦。

#### OATH HOTP

没用过，不知道。

#### 静态密码

顾名思义，输出一串固定的字符串作为密码。看起来一点卵用都没有（用个笔记本一装不就出来了嘛），但是这货在任何有密码输入框的地方都能使用，因此可以借助 Yubikey 输出一串非常难记而且预先随机生成的字符串来增强密码复杂度。

但是如果你觉得可以靠这个功能免手输密码，**那就完蛋了**，只要 Yubikey 遗失，后果不堪设想。所以静态密码的优雅且正确用法是：

- 输入一段自己的常规密码
- 在任何地方再插入 Yubikey 的随机密码（可以是在自己的密码前面、中间、后面，如果不想 Yubikey 自动按下回车键可以在 Yubikey Personalization Tool 里面的 Settings 取消 Yubikey 在最后输出 `\n` 的功能）

#### Challenge-Response

Challenge-Response 在 Yubikey 4 中有两种模式：Yubico OTP 和 HMAC-SHA1。Yubico OTP 可以在基于 Challenge-Response 的情况下以 Yubico OTP 的方式来做验证，需要网络连接到 YubiCloud；而 HMAC-SHA1 则是用于离线验证。

### PIV

[Yubikey 4][yk] 可以用作 PIV (FIPS 201-1) 规范的智能卡，可以用于一般智能卡用途，例如 SSH、Windows Active Directory 计算机登录，等等。但是，**Yubikey 的 PIV 并不是一个完全的 PIV 实现**，它与 OpenSC 兼容却不能完全使用某些功能，比如说不可以往卡里放文件。

使用 [Yubikey PIV Manager][ykpiv] 来管理 Yubikey 中的证书。[Yubikey 4][yk] 在 PIV 功能中共有 **18 个 Slot**：9a, 9c, 9d, 9e, 82-95, f9。**不要和 OTP 功能中的 Slot 混淆，它们是两个不同的概念**。其中：

- 9a 用于验证持卡人身份
- 9c 数字签名
- 9d 密钥管理，用于加密
- 9e 不知道
- 82-95 是过时的证书的 Slot
- f9 (>=4.3) 不知道

### OpenPGP Card

OpenPGP 卡配合 GPG 使用，用来存放 GPG 私钥（**拿不出来的**），并用作加密、签名用途。在 [Yubikey 4][yk] 此功能中，有 **3 个 Slot** 存放密钥：

- Signature（签名用途）
- Encryption（加 / 解密用途）
- Authentication（验证用途）

优雅的方法是生成不同用途的子钥，然后将子钥扔进卡里。**不要将主钥放进卡里**，因为主钥还承担人际关系的作用，一旦遗失 Yubikey，直接吊销子钥完事，而不必要吊销主钥然后重新建立 Web of Trust。

[yk]:           https://yubi.co/4
[pgpc]:         https://en.wikipedia.org/wiki/OpenPGP_card
[u2f]:          https://en.wikipedia.org/wiki/Universal_2nd_Factor
[fido]:         https://fidoalliance.org/
[yubico]:       https://yubico.com
[yubico-otp]:   https://developers.yubico.com/OTP/
[oath-hotp]:    https://developers.yubico.com/OATH/#_hotp
[static-pass]:  https://yubi.co/4
[chalresp]:     https://yubi.co/4
[ycloud]:       https://www.yubico.com/products/services-software/yubicloud/
[ykpiv]:        https://developers.yubico.com/PIV/
