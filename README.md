# nohost
nohost 是基于 [whistle](https://github.com/avwo/whistle) 实现的多用户多环境配置及抓包调试服务，不仅具备 whistle 的所有功能，并在 whistle 基础上扩展了一些功能，且支持多人多环境同时使用，主要用于部署在公共服务器上供整个部门（公司）的同事共同使用，具有以下功能：
1. 环境共享：前端无需配后台环境，后台无需配前端环境，其他人无需配任何环境
2. 抓包调试：远程实时抓包调试，支持各种 whistle 规则，以及通过链接分享抓包数据
3. 历史记录：可以把环境配置及抓包数据沉淀下来，供后续随时切换查看
4. 插件扩展：可以通过插件扩展实现诸如 [inspect](https://github.com/whistle-plugins/whistle.inspect)，[vase](https://github.com/whistle-plugins/whistle.vase)，[autosave](https://github.com/whistle-plugins/whistle.autosave) 等功能
5. 对外接口：提供对外接口，可供发布系统、CI等工具操作，实现自动化增删查改环境配置

![效果图](https://user-images.githubusercontent.com/11450939/40436253-28a90f28-5ee5-11e8-97a5-fd598e32e0df.gif)

# 一. 准备
安装 nohost 之前，建议先做好以下工作：

1. 准备一台服务器，假设IP为：10.222.2.200（以你自己的服务器为准）
2. 准备一个域名（以下假设为：imwebtest.oa.com），并把 DNS 指向上述服务器（10.222.2.200）
3. 收集涉及域名的证书对，只支持 `xxx.key` 和 `xxx.crt`（非必须，但建议用证书的证书，否则要么 nohost 里面无法查看 HTTPS 的内容，要么每个访问 nohost 的客户端都要安装一遍根证书）

> 申请域名的好处是可以直接用域名访问管理及账号页面，手机也可以通过域名设置代理访问 nohost，方便记忆及输入

# 二. 安装
首先，需要安装Node（建议使用最新的LTS版本）：[Node](https://nodejs.org/en/)

Node安装成功后，通过npm安装 `nohost`：
``` sh
npm i -g @nohost/server --registry=https://r.npm.taobao.org
```
安装完成后执行启动命令：
``` sh
n2 start
```
> nohost 的默认端口为 8080，如果需要自定义端口，可以通过 `n2 restart -p 80` 设置。
> 如果命令行提示没有对应命令，检查下系统环境变量 `PATH` 配置，看看 nohost 安装后生成的命令所在目录是否已添加到 `PATH`。

重启 `nohost`：
``` sh
n2 restart
```

停止 `nohost`：
``` sh
n2 stop
```

重置管理员账号：
``` sh
n2 restart --reset
```

# 三. 配置
安装启动成功后，打开管理员页面 `http://10.222.2.200:8080/admin.html#system/administrator`，输入默认用户名（`admin`）和密码（`123456`），打开系统配置后：
> 其中 `10.222.2.200` 表示nohost运行的服务器IP，具体根据实际 ServerIP 替换
1. 修改管理员的默认账号名和密码（**不建议使用默认账号及密码，如果忘记管理员账号名或密码，可以通过 `n2 restart --reset` 重置**）
2. 设置nohost的域名（将申请的域名填上，如果需要设置多个域名，可以通过逗号 `,` 分隔）
3. 上传涉及的 key 和证书（证书只支持 `.crt` 格式）

![admin](https://user-images.githubusercontent.com/11450939/69247822-0c010b00-0be6-11ea-8b03-5a0ae4b12c6e.gif)

**Note: 设置的域名 DNS 一定要指向该IP，否则可能出现不可用状态，上述配置会自动重启服务，建议避免频繁操作**

# 四. 访问
nohost 本身就是一个代理，可以直接配置浏览器或系统代理访问，也可以通过 Nginx反向代理访问，为方便大家使用，针对不同的人群可以使用不同的方案（以下用 `imwebtest.oa.com` 表示 nohost 的域名，具体域名需要自己申请及设置）。

#### 前端开发
前端开发建议使用最新版的 [whistle](https://github.com/avwo/whistle)，可以通过以下两种方式访问 nohost：

1. 直接在 whistle 上配置远程规则
    ``` txt
    @http://imwebtest.oa.com:8080/whistle.nohost/cgi-bin/plugin-rules
    ```
    > 上述配置表示 whistle 从 `http://imwebtest.oa.com:8080/whistle.nohost/cgi-bin/plugin-rules` 获取 nohost 的生成的入口规则，并且如果 nohost 规则有变会自动更新规则，这些规则是由 nohost 上传证书的域名及界面 `配置/入口配置` 配置的规则自动生成（具体参见后面的**配置**），这些规则可以自动过滤掉无关请求，只会把相关的请求转到nohost。

    当然这种直接手动配置在 whistle 上还不是最好的方式，更建议的方式是把这些规则集成到插件里面，这样开发者只需安装插件即可。
2. **【强烈推荐】** 集成 whistle 插件，具体参考：[https://github.com/nohosts/whistle.nohost-imweb/blob/master/dev.md](https://github.com/nohosts/whistle.nohost-imweb/blob/master/dev.md)

#### 后台开发
后台开发推荐使用 Chrome 的 [SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) 配置代理规则 （如上述代理配置 `imwebtest.oa.com` + `8080`），如果不想所有请求都转到 nohost，可以配置 SwitchyOmega 的自动切换或者用PAC脚本代替，也可以参考 `nohost-client` 打包一个客户端：[https://github.com/nohosts/nohost-client](https://github.com/nohosts/nohost-client)。手机端可以直接配代理，或者通过 VPN App 设置代理，如 iPhone 可以用 `detour`。

#### 其他人员
非开发人员尽量使用客户端、APP、或通过外网转发的方式，减少他们的接入成本，如何打包客户端参考：[https://github.com/nohosts/nohost-client](https://github.com/nohosts/nohost-client)；手机等同后台开发的配置方式。

#### 外网访问
一般 nohost 是部署在公司内网，外网是不可以直接访问，需要通过接入层（如：Nginx）转发，如何配置转发参见详细文档：https://nohosts.github.io/nohost/

# 五. 账号
安装好插件或配置好代理后，打开相关页面（这些页面的域名必须在上面上传证书里面，如果没有需要额外配置，具体参考下方 **配置** 说明），即可看到页面左下脚出现一个小圆点，点击小圆点可以进行切换环境：

![证书列表](https://user-images.githubusercontent.com/11450939/69306641-2d540c80-0c63-11ea-917f-f0fa0c88a222.png)

![whistle插件列表](https://user-images.githubusercontent.com/11450939/69324924-4ae59e00-0c84-11ea-994c-7c3914257470.png)

![打开页面](https://user-images.githubusercontent.com/11450939/69325133-9b5cfb80-0c84-11ea-8213-8c64d5365538.png)

![点击按钮](https://user-images.githubusercontent.com/11450939/69325935-ecb9ba80-0c85-11ea-8cca-3c73bee69fd6.png)

> 如果页面左下脚没出现小圆点，可以看下面 **配置** 说明
第一次打开小圆点只有一个 **正式环境**，需要管理员添加账号：
![添加账号](https://user-images.githubusercontent.com/11450939/69328087-93ec2100-0c89-11ea-83a6-c7914b3165a2.png)

添加完账号后，打开独立但环境选择页面 `http://imwebtest.oa.com:8080`：

![image](https://user-images.githubusercontent.com/11450939/69328692-a61a8f00-0c8a-11ea-8b14-61ecbaa47141.png)

# 六. 配置

# 七. 规则

# 八. 插件

**更多功能参见详细文档：[https://nohosts.github.io/nohost/](https://nohosts.github.io/nohost/)**


# License
[MIT](./LICENSE)