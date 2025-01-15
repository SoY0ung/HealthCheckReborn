# SRUN-SZU-Login

深圳大学教学区网络认证脚本和流程分析（深澜系统）

# 脚本

只包含Apple快捷指令和Linux Shell脚本。

## Apple 快捷指令脚本

支持 iOS 16及以上。

https://www.icloud.com/shortcuts/5b6f8f605fb641dab94e75be309468de

> ### 配置
>
> 首次添加快捷指令时，会弹出配置对话框，按提示输入用户名和密码即可运行。
>
> 如果你想重新配置，前往快捷指令详情页的“设置”选项中点击“自定义快捷指令...”以再次配置。
>
> 将快捷指令加入**"自动化"**可使其在连接到`SZU_WLAN`时自动运行。
>
> ### 隐私说明
>
> #### 权限信息
>
>    - 🌍 网页访问权限
>      需要访问认证服务器来完成登录。
>      此快捷指令会访问的站点如下：
>          ①net.szu.edu.cn
>    - 📣 通知权限
>      向你告知脚本运行结果。
>
> 你可以在快捷指令详情页面的隐私选项中管理快捷指令的权限。
>
> #### 密码存储
>
> 你的密码会明文保存在本地，仅在网络认证时会加密发送至认证服务器（net.szu.edu.cn），此快捷指令不会将你的密码发给任何第三方，明文密码只有你能看到，其他人无法读取。

## Shell脚本

基于bash。

https://github.com/SoY0ung/SZU-SRUN/blob/main/authSZU_office.sh

> 运行方式
>
> ```shell
> chmod +x /path/to/authSZU_office.sh
> bash /path/to/authSZU_office.sh
> ```
>
> Shell脚本使用了`openssl`, `base64`, `xxd`这三个组件。
>
> 在Linux发行版上运行需要安装 `openssl`。
>
> 在OpenWrt的路由器上运行需要安装`openssl-util`，`coreutils-base64`，`xxd`，`bash`。

# 流程分析

## 0 查询是否需要验证码（此过程可忽略）

> 检查此次登录是否需要验证码。
>
> **此过程可忽略**

**请求**

METHOD: **GET**

URL: https://net.szu.edu.cn/v2/srun_portal_captcha_image_info

Query:

 ```json
 user_name=登录用户名（学号或校园卡号）
 ip=当前设备的IP地址[该参数可不添加]
 ```

Example:

 https://net.szu.edu.cn/v2/srun_portal_captcha_image_info?user_name=123456&ip=127.0.0.1

**响应**

```json
{"code":0,"message":"Success","data":"0"}
```

当code为0且data不为1时则不需要验证码，直接进行认证操作。

## 1 获取Hash盐值

> 获取此次登录会话的Hash盐值

**请求**

METHOD: **GET**

URL: https://net.szu.edu.cn/cgi-bin/get_challenge

Query:

 ```json
callback=jsonp名称（不能留空，可以自定义，可以跟第二步的名称不一样）
username=登录用户名（学号或校园卡号）
ip=当前设备的IP地址[该参数可不添加]
_=13位时间戳[该参数可不添加]
 ```

Example:

https://net.szu.edu.cn/cgi-bin/get_challenge?callback=123abc&username=123456&ip=127.0.0.1&_=1735660800000

**响应**

```json
<callback>({
	"challenge": "abcdefghijklmnopqrstuvwxyz0123456789", //MD5的盐值
	"client_ip": "127.0.0.1",//当前设备的IP
	"ecode": 0,
	"error": "ok",
	"error_msg": "",
	"expire": "60",//盐值剩余有效时间（秒）
	"online_ip": "127.0.0.1",//与请求中的ip字段一致
	"res": "ok",
	"srun_ver": "SRunCGIAuthIntfSvr V1.18 B20241024",
	"st": 1735660800
})
```

需要在`expire`秒内将密码与盐值`challenge`进行MD5加密进行认证。

> **MD5加密方法**
>
> 密码和challenge采用HMAC-MD5进行加密，密码视为要加密的文本，challenge视为密钥，获得32位的小写密文
>
> Demo: https://www.toolhelper.cn/DigestAlgorithm/HMAC_MD5

## 2 发送认证请求

> 将若干参数发送给服务器请求登录

**请求**

METHOD: **GET**

URL: https://net.szu.edu.cn/cgi-bin/srun_portal

Query:

 ```json
callback=jsonp名称（不能留空，可以自定义，可以跟第一步的名称不一样）
action=login
username=登录用户名（学号或校园卡号）
password={MD5}XXXXXXXXXX（HMAC-MD5加密后的内容）
os=Windows 10[该参数可不添加]
name=Windows[该参数可不添加]
nas_ip=[该参数可不添加]
double_stack=0[该参数可不添加]
chksum=xxxxxxxxxxxxxxxxxxx（SHA1加密后的内容）
info={SRBX1}xxxxxxxxxyyyyyyyyyyzzzzzzzzz（基于Base64编码后的用户信息）
ac_id=12
ip=当前设备的IP地址（必须与设备ip一致）
n=200
type=1
captchaVal=[该参数可不添加]
_=13位时间戳[该参数可不添加]
 ```

Example:

https://net.szu.edu.cn/cgi-bin/srun_portal?callback=123abc&action=login&username=123456&password=%7BMD5%7DXXXXXXXXXX&os=Windows+9&name=Windows&nas_ip=&double_stack=0&chksum=xxxxxxxxxxxxxxxxxxx&info=%7BSRBX1%7Dxxxxxxxxxyyyyyyyyyyzzzzzzzzz&ac_id=12&ip=127.0.0.1&n=200&type=1&captchaVal=&_=1735660800000

**响应**

```json
<callback>({
	"ServerFlag": 0,
	"ServicesIntfServerIP": "127.0.0.1",
	"ServicesIntfServerPort": "8001",
	"access_token": "aaabbbcccddd",
	"checkout_date": 0,
	"client_ip": "127.0.0.1",//当前设备的IP
	"ecode": 0,
	"error": "ok",
	"error_msg": "",
	"online_ip": "127.0.0.1",//与请求中的ip字段一致
	"real_name": "",
	"remain_flux": 0,
	"remain_times": 0,
	"res": "ok",
	"srun_ver": "SRunCGIAuthIntfSvr V1.18 B20241024",
	"suc_msg": "login_ok",//login_ok代表登陆成功
	"sysver": "1.01.20241024",
	"username": "123456",//登录用户名（学号或校园卡号）
	"wallet_balance": 0
})
```

`suc_msg`的值为`login_ok`代表登陆成功。

### password字段

password字段由`{MD5}`和用HMAC-MD5加密后的密码拼接而成

### info和chksum字段

info字段由`{SRBX1}`和用户信息的编码结果拼接而成。

chksum字段是字符串`str`进行SHA1的密文。

> SHA1 Demo：https://www.jyshare.com/crypto/sha1/

字符串`str`由用户登录参数和info拼接而来。

### str的构成

`str = token + username + token + hmd5 + token + ac_id + token + ip + token + n + token + type + token + i`
`token`是第三步获取到的**Hash盐值**（即challenge），`username`是登录**用户名**（学号或校园卡号），`hmd5`是采用HMAC-MD5**加密后的密码**，`ac_id`是**12**（应该是套餐信息），`ip`是当前**设备的IP**，`n`是**200**（加密常量），`type`是**1**（加密常量），`i`是**info字段**（包含`{SRBX1}`）

### info字段的构造

`i = '{SRBX1}' + base64.encode(encode(info, token));`

**参考文章和代码**

https://www.ciduid.top/2022/1113/srun-login-analyze/

https://github.com/huxiaofan1223/jxnu_srun/tree/master/encryption

***Quick Look***

先将**原始info数据**（acid, env_ver, ip, password, username）和**盐值**（即challenge）进行一次编码`encode(info, token)`，然后将编码后的数据进行**base64编码**（注意这里的base64编码是修改过字母表的）

> 原始info数据是一个字符串
>
> `{"username":"123456","password":"pass","ip":"127.0.0.1","acid":"12","enc_ver":"srun_bx1"}`

下面会对函数`_encodeUserInfo`代码进行分析，写出具体过程。

#### 第一次编码

encode函数在开头调用了两次`s`函数，把`str`和`key`转成了一个数组`v[]`和`k[]`。

**分析s函数**

```js
function s(a, b) {	//a：字符串 b：布尔值，是否在数组末尾加入字符串的长度信息
    var c = a.length;
    var v = [];

    for (var i = 0; i < c; i += 4) { //每次迭代处理 4 个字符
    	v[i >> 2] = a.charCodeAt(i) | a.charCodeAt(i + 1) << 8 
        			| a.charCodeAt(i + 2) << 16 | a.charCodeAt(i + 3) << 24;
    }

    if (b) v[v.length] = c; //将长度插入到数组末尾
    return v;
}
```

大概逻辑为从头开始遍历字符串数组a，每次拿四个元素（不足四个时有多少拿多少），把这四个元素用二进制来看待，每个元素都是8bit，四个8bit正好组成一个int(32bit)输出。组成方法为第一个元素放到低8位，第二个放到中低8位，第三个放到中高8位，第四个放到高8位。

C++程序为：

```cpp
vector<int> s(string str, bool addLen) {
    int len=str.length();
    int group=len/4+(len%4?1:0); //每四个一组，一共group组
    vector<int> v(group,0);
    /*
    for (int i = 0; i < len; i += 4)  //每次迭代处理 4 个字符
    	v[i/4] = str[i] + str[i+1]*256 + str[i+2]*65536 + str[i+3]*16777216;
    */
    for (int i = 0; i < group; i++){  //每组进行拼接
        for(int j=0; j<4; j++) { //每组拿4个
            if(i*4+j<len) v[i]+=str[i*4+j]*pow(256,j);
            else break; //数组越界，跳过
        }
    }    
    if(addLen) v.push_back(len); //将长度插入到数组末尾
    return v;
}
```

---

对`v[]`和`k[]`进行处理后送入`l`函数

**分析l函数**

```js
function l(a, b) { // b：布尔值，对数组末尾加入字符串长度信息的串进行还原
    var d = a.length;
    var c = d - 1 << 2;

    if (b) {
        var m = a[d - 1]; //获取最后一个元素
        if (m < c - 3 || m > c) return null; //非法串
        c = m; //c是原字符串的长度
    }

    for (var i = 0; i < d; i++) {
    	a[i] = String.fromCharCode(a[i] & 0xff, a[i] >>> 8 & 0xff, a[i] >>> 16 & 0xff, a[i] >>> 24 & 0xff);
    }

    return b ? a.join('').substring(0, c) : a.join('');
}
```

`l`函数是`s`函数的逆操作，把每个32bit的元素拆分成4个8bit的元素。遍历数组a，将每个元素拆分成4个元素，拆分方法为当前元素的低8位作为第一个元素，当前元素的中低8位作为第二个元素，当前元素的中高8位作为第三个元素，当前元素的高8位作为第四个元素。

C++程序为：

```cpp
vector<int> l(vector<int> a, bool withLen) {
    int len=a.size();
    int guessLen=(len-1)*4;
    vector<int> v;
	if(withLen) {
        int lenElement=a[len-1];
        if(lenElement<guessLen-3 || lenElement>guessLen) return v;
        guessLen=lenElement;
    }
    for (int i = 0; i < len; i++) {
        v.push_back(a[i] & 0xff);
        v.push_back(a[i] >> 8 & 0xff);
        v.push_back(a[i] >> 16 & 0xff);
        v.push_back(a[i] >> 24 & 0xff);
    }
  
    if(withLen) v.pop_back(); //去掉数组末尾的代表数组长度的元素
    return v;
}
```

---

接下来就是`encode`的剩余部分，直接照搬思路。

**encode代码**

```js
function encode(str, key) {
    if (str === '') return '';
    var v = s(str, true);
    var k = s(key, false);
    if (k.length < 4) k.length = 4;
    var n = v.length - 1,
    z = v[n],
    y = v[0],
    c = 0x86014019 | 0x183639A0,
    m,
    e,
    p,
    q = Math.floor(6 + 52 / (n + 1)),
    d = 0;

    while (0 < q--) {
        d = d + c & (0x8CE0D9BF | 0x731F2640);
        e = d >>> 2 & 3;

        for (p = 0; p < n; p++) {
            y = v[p + 1];
            m = z >>> 5 ^ y << 2;
            m += y >>> 3 ^ z << 4 ^ (d ^ y);
            m += k[p & 3 ^ e] ^ z;
            z = v[p] = v[p] + m & (0xEFB8D130 | 0x10472ECF);
        }

        y = v[0];
        m = z >>> 5 ^ y << 2;
        m += y >>> 3 ^ z << 4 ^ (d ^ y);
        m += k[p & 3 ^ e] ^ z;
        z = v[n] = v[n] + m & (0xBB390742 | 0x44C6F8BD);
    }

    return l(v, false);
}
```

按照算法可以写出C++代码

```cpp
vector<int> encode(string str, string key){
    vector<int> strArr= s(str,1);
    vector<int> keyArr= s(key,0);
    if(keyArr.size()<4) {
        for(int i=0; i<(4-keyArr.size()); i++) {
			keyArr.push_back(0);
		}
    }
    unsigned int n=strArr.size()-1;
    unsigned int z=strArr[n];
    unsigned int y=strArr[0];
    unsigned int c = 0x86014019 | 0x183639A0;
    unsigned int m=0, e=0, p=0, iter=6+52/(n+1), d=0;

	while(1){
        iter--;
        d = (d + c) & (0x8CE0D9BF | 0x731F2640);
        e = d >> 2 & 3;
        for(p=0; p<n; p++){
			y = strArr[p+1];
			m = z >> 5 ^ y << 2;
			m += (y>>3 ^ z<<4) ^ (d ^ y);
			m += (unsigned int)keyArr[(p&3)^e] ^ z;
			strArr[p] = (strArr[p] + m) & (0xEFB8D130|0x10472ECF);
            // cout<<"strArr["<<p<<"] "<<strArr[p]<<",";
			z = strArr[p];
		}
        cout<<endl;
        y = strArr[0];
		m = z >> 5 ^ y << 2;
		m += (y>>3 ^ z<<4) ^ (d ^ y);
		m += keyArr[(n&3)^e] ^ z;
		strArr[n] = (strArr[n] + m) & (0xBB390742 | 0x44C6F8BD);
		z = strArr[n];
		if (0 >= iter){
			break;
		}

    }
    return l(strArr, 0);
}
```

#### 第二次编码

第二次编码是对第一次编码后的字符串（或者说数组）进行一次自定义的base64编码。自定义的意思是base64的字母表被换了顺序。

`base64.setAlpha('LVoJPiCN2R8G90yg+hmFHuacZ1OWMnrsSTXkYpUq/3dlbfKwv6xztjI7DeBE45QA');`

用C++实现

```cpp
static const char* base64_char = {
             "LVoJPiCN2R8G90yg+hmFHuacZ1"
             "OWMnrsSTXkYpUq/3dlbfKwv6xz"
             "tjI7DeBE45"
             "QA"};

std::string base64_encode( char const* bytes_to_encode, size_t in_len, bool url=false) {
    size_t len_encoded = (in_len +2) / 3 * 4;
    unsigned char trailing_char = '=';
    const char* base64_chars_ = base64_char;

    std::string ret;
    ret.reserve(len_encoded);

    unsigned int pos = 0;
    while (pos < in_len) {
        ret.push_back(base64_chars_[(bytes_to_encode[pos + 0] & 0xfc) >> 2]);

        if (pos+1 < in_len) {
           ret.push_back(base64_chars_[((bytes_to_encode[pos + 0] & 0x03) << 4) + ((bytes_to_encode[pos + 1] & 0xf0) >> 4)]);

           if (pos+2 < in_len) {
              ret.push_back(base64_chars_[((bytes_to_encode[pos + 1] & 0x0f) << 2) + ((bytes_to_encode[pos + 2] & 0xc0) >> 6)]);
              ret.push_back(base64_chars_[  bytes_to_encode[pos + 2] & 0x3f]);
           }
           else {
              ret.push_back(base64_chars_[(bytes_to_encode[pos + 1] & 0x0f) << 2]);
              ret.push_back(trailing_char);
           }
        }
        else {
            ret.push_back(base64_chars_[(bytes_to_encode[pos + 0] & 0x03) << 4]);
            ret.push_back(trailing_char);
            ret.push_back(trailing_char);
        }

        pos += 3;
    }
    return ret;
}
//base64_encode(str.c_str(),str.length());
```

​                                                                                                                               

