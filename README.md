<div align="center">
  <h1>MagicMihomo</h1>
  <p><strong>一份面向 MagicNet 与 Android TUN 场景的 Mihomo 配置基线。</strong></p>
  <p>DNS 分流 · TUN · 规则集 · 节点分组 · Zashboard</p>

  [![Stars](https://img.shields.io/github/stars/LIghtJUNction/MagicMihomo?style=flat-square)](https://github.com/LIghtJUNction/MagicMihomo/stargazers)
  [![Last commit](https://img.shields.io/github/last-commit/LIghtJUNction/MagicMihomo?style=flat-square)](https://github.com/LIghtJUNction/MagicMihomo/commits/main)
  [![License](https://img.shields.io/github/license/LIghtJUNction/MagicMihomo?style=flat-square)](LICENSE)
</div>

## 本质上是什么

这不是一个代理客户端，也不包含可直接使用的节点。它把一套 Mihomo 运行策略拆成四部分：

```text
proxies/*.yaml   节点输入
       ↓
config.yaml      DNS、TUN、嗅探和策略组
       ↓
ruleset/*.yaml   域名与 IP 分类
       ↓
zashboard/       可视化控制面板
```

仓库提供的是**可编辑的默认配置**。你仍需放入自己的节点、检查端口和系统权限，并在目标设备上验证实际网络行为。

## 先完成这一步

`config.yaml` 默认读取三个本地提供器：

```yaml
proxy-providers:
  premium_a:
    type: file
    path: ./proxies/A.yaml
  premium_b:
    type: file
    path: ./proxies/B.yaml
  premium_c:
    type: file
    path: ./proxies/C.yaml
```

仓库中的 `A.yaml`、`B.yaml`、`C.yaml` 只是空占位文件。将自己的 Mihomo 节点列表写入这些文件，或者把 `proxy-providers` 改成你实际使用的本地文件或远程提供器。

不要把订阅链接、认证信息或私人节点提交到公开仓库。

## 配置重点

| 模块 | 当前策略 |
| --- | --- |
| DNS | Fake-IP、国内/海外 DoH 分流、私有网络真实解析、广告域名拒绝 |
| TUN | `gvisor` 栈、自动路由、严格路由、DNS 劫持、私网与 VPN 应用排除 |
| 嗅探 | TLS / HTTP / QUIC，跳过局域网、Apple、Microsoft、Tailscale 等敏感域名 |
| 节点组 | 美国、日本、香港、台湾地区及其他节点自动测速/回退 |
| 业务组 | AI、游戏、流媒体、Telegram、Apple、Google、Microsoft、广告等 |
| 规则外流量 | 默认直连，避免未命中的国内应用被意外代理 |

配置面向 Android KernelSU/Magisk 一类 TUN 场景，因此默认使用 `gvisor`，并排除 Tailscale、WireGuard、OpenVPN、ZeroTier 和常见代理客户端，降低路由环路风险。

## 使用方式

```bash
git clone https://github.com/LIghtJUNction/MagicMihomo.git
cd MagicMihomo
```

然后：

1. 填充或替换 `proxies/*.yaml`。
2. 检查 `proxy-providers` 与 `providers-default` 是否引用了全部目标提供器。
3. 按设备环境调整 TUN 接口、排除应用、DNS 监听地址和规则组默认值。
4. 使用你安装的 Mihomo 版本做配置检查。
5. 在测试设备上逐项验证 DNS、IPv4/IPv6、局域网、Tailscale、应用分流与断网恢复。

常见 Mihomo 构建可用下面的形式检查配置；具体参数以你使用的二进制帮助信息为准：

```bash
mihomo -t -f config.yaml
```

配置检查通过只说明语法和静态引用可被当前核心接受，不代表节点可用、规则符合你的网络，或所有 Android 流量都已正确接管。

## 目录

```text
config.yaml       主配置与策略组
proxies/          私有节点文件的本地占位目录
ruleset/          自动生成的域名/IP 规则集
zashboard/        Mihomo Web 控制面板静态文件
.github/workflows/update.yml
                  每日更新规则集的工作流
```

## 规则集更新

`Update All` 工作流每天抓取公开上游规则，转换为 Mihomo YAML，并提交到 `ruleset/`。来源包括 Loyalsoldier、felixonmars 和其他在工作流中明确列出的地址。

自动更新的本质只是：

```text
下载上游文本 → 按规则转换 → 检查输出非空 → 覆盖 ruleset → 提交
```

上游内容和分类可能变化。关键业务部署应固定提交、审阅差异，并在升级后重新验证，不要把“自动更新成功”当作“网络策略一定正确”。

## 与 MagicNet 的关系

[MagicNet](https://github.com/LIghtJUNction/MagicNet) 是完整的 Android root 网络控制平台；MagicMihomo 只维护其中可独立使用和审阅的 Mihomo 配置、规则与面板材料。两者版本和发布周期相互独立。

## License

[MIT](LICENSE)
