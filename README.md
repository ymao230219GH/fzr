执行 Github Actions


GitHub 仓库通过update-worker.yml 会自动下载最新的 BPB 源代码
.last_version：记录更新版本信息
_worker.js：最新BPB 代码
三、Cloudflare 部署
1. 登录 Cloudflare，创建 Pages 部署image-20250322100735664
在 Cloudflare 控制台中进入 Workers 和 Pages，选择 Pages 部署。


连接到你的 Github 仓库，选择刚才新建的 BPB 项目仓库，然后点击开始部署。
2. 绑定自定义域名（可选）


在 Pages 项目的 自定义域选项卡，点击设置自定义域。
3. 设置变量image-20250322101207119
部署成功后，在 Pages 项目界面点击 设置 -> 变量和机密，添加以下变量：

UUID：使用 UUID 生成器 随机生成一个新的 UUID。
PROXY_IP：填写代理 IP 地址，可从 随机代理 IP 站点 获取，或使用优选域名（例如 cdn-b100.xn--b6gac.eu.org）。
TR_PASS：填写一个复杂字符串，作为密码。
4. 绑定 KV 命名空间image-20250322103724074
创建 KV:点击左侧存储和数据库，再选择 KV,然后创建一个新的 KV 命名空间

注:名称自定义但不能包含“bpb”image-20250322103833926

绑定 KV:回到创建的 Pages 界面。点击 设置 -> 【绑定】，点击添加，选择添加 KV 命名空间

注:变量名称只能填写“kv”(小写)

5. 重试部署 Pages


返回 Pages 项目,找到右侧“…”,点击重试部署
四、BPB 面板设置
1. 验证部署是否成功


打开浏览器输入:https://[自定义域名]或者你的项目地址,后面加上/panel,检查是否能正常访问BPB面板
2. 修改 BPB 面板密码


第一次打开 BPB 面板会提示修改密码,请设置一个复杂密码,避免被盗用
3. 配置 BPB 面板参数
下面是BPB面板的参数解析，给大家做参考，请自行测试其效果。
🌐 VLESS - Trojan基础设置

项目	说明	建议填写
Remote DNS	远程 DNS 请求地址	https://8.8.8.8/dns-query
Local DNS	本地 DNS 服务器	8.8.8.8
Fake DNS	虚假 DNS 功能	Enabled
Proxy IPs / Domains 获取地址：点击访问	代理 IP/域名	141.147.156.68、cdn-xx-b6gac.acu.org
Chain Proxy	链式代理	（空，默认）
Clean IPs / Domains	清洁 IP/域名	104.17.212.246、104.19.19.167
Clean IP Scanner	下载扫描器按钮	点击 Download Scanner
IPv6	是否启用 IPv6	Enabled
Custom CDN Addrs / Host / SNI	CDN 自定义字段	空
Best Interval	最佳间隔值	30
Protocols	协议选择	VLESS 和 Trojan 都勾选
TLS Ports	TLS 端口	勾选：443、8443、2053、2083、2087、2096
✂️ Fragment 分片设置
项目	说明	建议填写
Length	分片长度范围	100 - 200
Interval	间隔时间范围	1 - 1
Packets	分片类型	tlsHello
🔄 Warp General 设置
项目	说明	建议填写
Endpoints	Warp 端点	engage.cloudflareclient.com:2408
Scan Endpoint	脚本按钮	点击 “Copy Script”
Fake DNS	虚假 DNS	Disabled
IPv6	启用 IPv6	Enabled
Warp Configs	配置更新	点击 “Update”
Best Interval	间隔设置	30
🚀 Warp PRO 设置（各客户端）
🌀 Hiddify / MahsaNG / NikaNG
项目	说明	建议填写
Hiddify Mode	模式选择	m4
NikaNG Mode	传输协议	quic
Noise Count	噪声包数量范围	10 - 15
Noise Size	噪声包大小范围	5 - 10
Noise Delay	噪声延迟范围	1 - 1
🧩 Clash / Amnezia
项目	说明	建议填写
Noise Count	噪声包数量	5
Noise Size	噪声包大小范围	50 - 100
🔊 v2rayNG / v2rayN
项目	说明	建议填写
v2ray Mode	模式	Random
Noise Packet	噪声包范围	50 - 100
Noise Delay	延迟时间范围	1 - 1
Noise Count	噪声数量	5
🧭 Routing Rules 路由规则
🟩 预设规则（Preset Rules）
类型	项目	勾选建议
Bypass	绕过区域	✅ LAN、Iran、Russia、ChatGPT（China 可选）
Block	阻止内容	✅ Ads、Porn（QUIC 可选）
🔧 自定义规则（Custom Rules）
项目	说明
Bypass IPs / Domains	自定义绕过 IP/域名
Block IPs / Domains	自定义阻止 IP/域名
全部设置好，点击 APPLY SETTINGS 保存 BPB 面板配置
五、VPN 节点部署完成
1. 导出节点订阅链接
根据你所使用的代理软件，点击对应的 COPY SUB 按钮，复制 BPB 面板生成的订阅链接。

2. V2rayN 客户端导入节点订阅链接并使用
打开 V2rayN，进入【订阅分组】->【订阅分组设置】->【添加】,将订阅链接粘贴进去

点击【订阅分组】->【更新全部订阅(不通过代理)】，获取最新节点信息

测试节点延迟,确认节点有效后,开启系统代理,即可使用 VPN

现在 BPB 面板 VPN 部署全部结束,通过以上步骤,你可以利用 Cloudflare 和 BPB Panel 搭建一个永久免费 VPN,同时通过对 BPB 源代码进行加密混淆生成专属混淆代码,成功绕过 Cloudflare 的审查,解决 1101 报错问题,本期教程不仅支持 singbox-core 和 xray-core 等跨平台客户端等配置,还实现了永久免费节点订阅,满足大多数用户的使用需求,大家可以一起部署折腾,有什么问题,请在评论区留言,一起研究学习.
