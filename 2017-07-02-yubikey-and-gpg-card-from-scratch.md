# 从零开始的 GPG key 与 Yubikey

由于之前的 keytocard 事故与 expert mode 的重复 sign/revoke uid 事故，致使窝最终选择创建个新的 key，顺便来个一条龙避免又掉进之前弄出来的坑。

本文介绍从零开始的 GPG key 创建和 Yubikey 配置，以创建一个包含多个 subkey 的 master key，并且将 subkey 塞进 Yubikey 使用为例。

由于 gpg 输出信息非常多，文中做了一定删减（结果还是很长）。

## 创建 gpg key


准备干净的 gpg 目录，推荐使用一个外部的比如 U 盘一类的设备软链接到 `~/.gnupg` 来用。

```sh
# 操作 .gnupg 目录前先关闭 gpg-agent
gpgconf --kill gpg-agent

mv ~/.gnupg .gnupg.orig
mkdir ~/.gnupg
```

### 创建 master key

`--expert` 用于自定义 master key 的权能，这里只给 master key 分配了 Certify 权能。

```
~> gpg --expert --full-generate-key

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Sign Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Jiahao Guo
Email address: archer@frantic1048.com
Comment: 
You selected this USER-ID:
    "Jiahao Guo <archer@frantic1048.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/chino/.gnupg/trustdb.gpg: trustdb created
gpg: key 96F3CA6256A95C51 marked as ultimately trusted
gpg: directory '/home/chino/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/chino/.gnupg/openpgp-revocs.d/6F6E9653B88BA07F5EED1B0096F3CA6256A95C51.rev'
public and secret key created and signed.

pub   ed25519 2017-08-02 [C]
      6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
      6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
uid                      Jiahao Guo <archer@frantic1048.com>
```

如上输出所示，吊销用的证书也已经自动生成了，并且里面包含了使用说明，和防止误用的额外额特殊字符。

添加 subkeys，以及（可选）额外的 uid：

```
~> gpg --expert --edit-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51

# 添加签名用 key
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Key is valid for? (0) 3y
Is this correct? (y/N) y
Really create? (y/N) y

# 添加加密用 key
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 6
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Key is valid for? (0) 3y
Is this correct? (y/N) y
Really create? (y/N) y

# 添加认证用 key
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 8

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Sign Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? e

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? a

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Authenticate 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Key is valid for? (0) 3y
Is this correct? (y/N) y
Really create? (y/N) y

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>

# 添加更多的 uid
gpg> adduid 
Real name: frantic1048
Email address: i@frantic1048.com
Comment: 
You selected this USER-ID:
    "frantic1048 <i@frantic1048.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1)  Jiahao Guo <archer@frantic1048.com>
[ unknown] (2). frantic1048 <i@frantic1048.com>

# 选定新的 uid 并信任它
gpg> uid 2

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1)  Jiahao Guo <archer@frantic1048.com>
[ unknown] (2)* frantic1048 <i@frantic1048.com>

gpg> trust
sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1)  Jiahao Guo <archer@frantic1048.com>
[ unknown] (2)* frantic1048 <i@frantic1048.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1)  Jiahao Guo <archer@frantic1048.com>
[ unknown] (2)* frantic1048 <i@frantic1048.com>

gpg> save
```

由于新添加的 uid 会默认成为新的 primary uid，根据需要，可以重新调整最初的 uid 作为 primary uid。


```
~> gpg --expert --edit-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). frantic1048 <i@frantic1048.com>
[ultimate] (2)  Jiahao Guo <archer@frantic1048.com>

# 选定要作为 primary uid 的 uid
gpg> uid 2

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). frantic1048 <i@frantic1048.com>
[ultimate] (2)* Jiahao Guo <archer@frantic1048.com>

# 设置为 primary
gpg> primary

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1)  frantic1048 <i@frantic1048.com>
[ultimate] (2)* Jiahao Guo <archer@frantic1048.com>

gpg> save
```

如上操作后，gpg key 的创建就算完成了，可以通过 `gpg -K` 查看总览。

```
~> gpg -K
gpg: WARNING: unsafe permissions on homedir '/home/chino/.gnupg'
/home/chino/.gnupg/pubring.kbx
------------------------------
sec   ed25519 2017-08-02 [C]
      6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
uid           [ultimate] Jiahao Guo <archer@frantic1048.com>
uid           [ultimate] frantic1048 <i@frantic1048.com>
ssb   rsa4096 2017-08-02 [S] [expires: 2020-08-01]
ssb   rsa4096 2017-08-02 [E] [expires: 2020-08-01]
ssb   rsa4096 2017-08-02 [A] [expires: 2020-08-01]
```

### 备份 secret key

由于后面进行的将 subkey 写入 Yubikey 的过程是破坏性的操作，务必在这个时候备份 secret key。顺便也导出一份 pub key。

将 secret key 放到足够稳妥的地方保存。

```
gpg --export-secret-keys archer@frantic1048.com > /mnt/rabbithouse/sec.gpg
gpg --export archer@frantic1048.com > /mnt/rabbithouse/pub.gpg
```

为了验证备份成功了，就好比保存两次以示放心（x，重新导入一次 secret key。

```
gpg --delete-secret-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
gpg --import < /mnt/rabbithouse/sec.gpg
```

## 配置 Yubikey

在 Arch 环境下先安装 yubikey-personalization-gui 包。

### 设定 Yubikey 工作模式

详见 `man ykpersonalize`。

```
ykpersonalize -m82
```

### 设置密码

```
gpg2 --card-edit

# 启用管理员命令
gpg/card> admin
Admin commands are allowed

# 修改 PIN 和 Admin PIN
# 默认 PIN: 123456
# 默认 Admin PIN: 12345678
gpg/card> passwd

# 签名时强制需要 PIN
gpg/card> forcesig

gpg/card> quit
```


### 写入 GPG subkey

```
~> gpg --expert --edit-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51

Secret key is available.

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

# 选定要写入的 subkey
gpg> key 1

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb* rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

# 将选定的 subkey 写入 Yubikey
gpg> keytocard 
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb* rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

# 取消选择不再需要操作的 subkey
gpg> key 1

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

# 选择另一个需要写入的 subkey
gpg> key 2

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb* rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

gpg> keytocard 
Please select where to store the key:
   (2) Encryption key
Your selection? 2

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb* rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

gpg> key 2

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb  rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

gpg> key 3

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb* rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

gpg> keytocard 
Please select where to store the key:
   (3) Authentication key
Your selection? 3

sec  ed25519/96F3CA6256A95C51
     created: 2017-08-02  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/406E81EE26FA324A
     created: 2017-08-02  expires: 2020-08-01  usage: S   
ssb  rsa4096/C21F4C84D0760AA8
     created: 2017-08-02  expires: 2020-08-01  usage: E   
ssb* rsa4096/33B7F4402CC030CA
     created: 2017-08-02  expires: 2020-08-01  usage: A   
[ultimate] (1). Jiahao Guo <archer@frantic1048.com>
[ultimate] (2)  frantic1048 <i@frantic1048.com>

gpg> save
~> gpg -K
gpg: WARNING: unsafe permissions on homedir '/home/chino/.gnupg'
/home/chino/.gnupg/pubring.kbx
------------------------------
sec   ed25519 2017-08-02 [C]
      6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
uid           [ultimate] Jiahao Guo <archer@frantic1048.com>
uid           [ultimate] frantic1048 <i@frantic1048.com>
ssb>  rsa4096 2017-08-02 [S] [expires: 2020-08-01]
ssb>  rsa4096 2017-08-02 [E] [expires: 2020-08-01]
ssb>  rsa4096 2017-08-02 [A] [expires: 2020-08-01]
```

ssb 后面的 `>` 即表示这个 subkey 已经在 Yubikey 上了。

## 发布 GPG key

将 key 发送到 keyserver 上，这里以 sks.ustclug.org 为例，常用的还有 pgp.mit.edu，hkps.pool.sks-keyservers.net。

```
gpg --keyserver hkps://sks.ustclug.org --send-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
```

此时可以移除包含 master key 的 .gnupg 目录，恢复开始的 .gnupg.orig 目录为 .gnupg。注意不要误删之前备份的 secret key!

```
gpgconf --kill gpg-agent
rm -r ~/.gnupg
```

## 使用 Yubikey

### 准备

为 Yubikey 设置 pubkey url，顺便导入 pub key。

```
gpg --card-edit

gpg/card> admin
Admin commands are allowed

gpg/card> url
URL to retrieve key: http://sks.ustclug.org/pks/lookup?op=get&search=0x6F6E9653B88BA07F5EED1B0096F3CA6256A95C51

gpg/card> fetch

gpg/card> quit
```

### 更多

#### 用 authentication subkey 作 SSH 认证

gpg 有直接的命令导出 SSH 吃的 pub key：

```
gpg --export-ssh-key 6F6E9653B88BA07F5EED1B0096F3CA6256A95C51
```


#### 令 ssh 与 gpg-agent 协作（代替 ssh-agent）

参见 https://wiki.archlinux.org/index.php/GnuPG#SSH_agent

