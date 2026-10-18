# n8n服务器推荐：内存怎么选、一年真实花多少钱，附 Docker 部署与安全加固要点

n8n 自托管版本身免费开源（MIT 协议），真正要花钱的只有一台 7×24 小时在线的服务器。结论先给：个人自动化工作流，选 Vultr 的 Cloud Compute 2GB 套餐起步——共享 CPU Regular 档每月 10 美元，High Performance 档每月 12 美元；工作流里带 AI 节点、并发任务多，就上 4GB 档。这笔开销比 n8n 官方云托管每月 20 美元起的报价低一截，数据还完整握在自己手里。

> **选购速览**：绝大多数自建 n8n 的用户，选 Vultr Cloud Compute 共享 CPU 的 2GB 或 4GB 档就够了，月费 10–24 美元，按小时计费、随时销毁。机房优先挑离你近的亚太节点。[👉 查看 Vultr 全部套餐与机房分布](https://www.vultr.com/pricing/?ref=7268283)

## 跑 n8n 到底需要什么配置

内存是唯一值得认真挑的指标：2GB 是底线，4GB 用起来踏实，其余配置都不用纠结。

多数 n8n 部署教程给出的口径一致——处理复杂工作负载至少 2GB 内存，推荐 4GB 以上。工作流执行历史、节点之间的数据转换、AI 类节点都吃内存，1GB 机器大概率在某个深夜的批量任务里给你溢出一次。

CPU 反而最不用操心。n8n 的负载是间歇性的：工作流被 webhook 或定时器触发时跑几秒，其余时间空转。2 vCPU 覆盖绝大多数场景，有部署教程实测 2核2GB 就够日常使用。

带宽是最容易被带偏的一项。n8n 在网络上跑的是 API 请求和 webhook 回调，一条消息几百 KB 封顶，Vultr 入门档附带的 2TB 月流量正常用一年都用不完。哪家宣传页把 8TB、16TB 流量当核心卖点，对跑 n8n 来说都只是个数字。

磁盘只有一个要求：SSD。n8n 默认用 SQLite，执行日志会持续累积，机械盘会让定时任务明显变慢。

## 为什么把 Vultr 放进 n8n服务器推荐清单

按小时计费、免费防火墙、机房多——这三点对 n8n 用户都是实打实的需求。

**按小时计费**。Vultr 实例按小时扣费，每月封顶按套餐价结算，没有年约。想先验证 n8n 适不适合自己，开一台小机器跑一个下午，成本几毛钱，不满意直接销毁。对比动辄按月按年预付的促销套餐，这种试错方式几乎没有沉没成本。[👉 按小时开一台 Vultr 先试跑](https://bit.ly/VulTR)

**免费防火墙**。下文会提到 Censys 的最新数据：暴露在公网的 n8n 已经超过 11 万台，其中 76% 带着已知高危漏洞在运行。Vultr 的 Cloud Firewall 在控制台免费开启，按组下发规则，5678 端口可以直接对公网关死，这一步能挡掉绝大多数扫描。

**机房与付款**。全球 30 多个机房可选，国内用户选东京、首尔、新加坡这类亚太节点，延迟明显好于美国节点；付款支持信用卡、PayPal 和支付宝，不需要海外卡。

**运维配套**。自动备份按实例价格加收 20%；升级前可以先打快照；套餐随时升档，不用换机器迁移数据。

## Vultr Cloud Compute 全套餐价格表

跑 n8n 的主力是 Cloud Compute 共享 CPU 线，全部档位如下，点击套餐名进入官方定价页：

| 套餐 | vCPU | 内存 | SSD | 月流量 | 月付 |
| --- | --- | --- | --- | --- | --- |
| [Regular 1GB](https://www.vultr.com/pricing/?ref=7268283) | 1 | 1GB | 25GB | 1TB | $5 |
| [Regular 2GB](https://www.vultr.com/pricing/?ref=7268283) | 1 | 2GB | 55GB | 2TB | $10 |
| [Regular 4GB](https://www.vultr.com/pricing/?ref=7268283) | 2 | 4GB | 80GB | 3TB | $20 |
| [Regular 8GB](https://www.vultr.com/pricing/?ref=7268283) | 4 | 8GB | 160GB | 4TB | $40 |
| [Regular 16GB](https://www.vultr.com/pricing/?ref=7268283) | 8 | 16GB | 320GB | 5TB | $80 |
| [High Performance 1GB](https://www.vultr.com/pricing/?ref=7268283) | 1 | 1GB | 25GB | 1TB | $6 |
| [High Performance 2GB](https://www.vultr.com/pricing/?ref=7268283) | 1 | 2GB | 55GB | 2TB | $12 |
| [High Performance 4GB](https://www.vultr.com/pricing/?ref=7268283) | 2 | 4GB | 80GB | 3TB | $24 |
| [High Performance 8GB](https://www.vultr.com/pricing/?ref=7268283) | 4 | 8GB | 160GB | 4TB | $48 |
| [High Performance 16GB](https://www.vultr.com/pricing/?ref=7268283) | 8 | 16GB | 320GB | 5TB | $96 |

几个阅读提示：

- High Performance 档采用更新的处理器和存储，价格统一比 Regular 档贵 20%，单核性能更强。
- n8n 大部分时间在等 API 响应，CPU 差异感知不强。同预算下选内存更大的 Regular，还是单核更快的 High Performance，看你工作流是“节点多”还是“单次计算重”。
- 1GB 两档低于 2GB 底线，只适合做部署演练。
- Vultr 还有专属 vCPU、裸金属和 GPU 产品线，单机跑 n8n 用不到，只有搭队列模式集群才需要，档位和价格以官网定价页为准。

## 一年真实成本对比

拿 n8n Cloud 当基准最直观：官方托管每月 20 美元起（有 14 天免费试用），一年 240 美元起。同样预算，Vultr High Performance 4GB 跑满一整年还剩 48 美元。

| 方案 | 配置参考 | 月费 | 一年费用 | 计费方式 |
| --- | --- | --- | --- | --- |
| n8n Cloud（官方托管） | 官方运维，免部署 | $20 起 | $240 起 | 月付，14 天试用 |
| Hostinger KVM 2（促销价） | 2核 / 8GB / 100GB NVMe | 约 $6.5–7 | 视续费周期 | 促销价，有 n8n 一键模板 |
| Kamatera（2GB 档） | 自定义配置 | $6 | $72 | 随开随用，30 天免费试用 |
| CloudCone（年付促销） | 2核 / 2GB | 约 $2.5 | 约 $30 | 年付促销价 |
| Vultr Regular 2GB | 1核 / 2GB / 55GB SSD | $10 | $120 | 按小时计费，无合约 |
| Vultr High Performance 4GB | 2核 / 4GB / 80GB SSD | $24 | $288 | 按小时计费，无合约 |

注：Hostinger、CloudCone 的价格来自第三方测评文章的促销口径，续费价以各自官方页面为准。

单看月费，Vultr 排不进最便宜的那档——Hostinger 促销期 8GB 内存卖 6.5 美元上下，确实狠。但那类价格通常绑促销周期，Vultr 的差异化在别处：按小时计费意味着服务器可以“用的时候才存在”；免费防火墙和快照在控制台一键开通，不用另买安全服务。只为省每年几十美元，选谁都不亏；要的是随时销毁、随时重建的灵活度，就 [👉 对比 Vultr 各档位当前定价](https://www.vultr.com/pricing/?ref=7268283) 再做决定。

## 从开通到打开工作台：五步部署

部署难度被不少教程夸大了，跟着走半小时内能见到工作台。

1. **开通实例**。注册 Vultr，选 Cloud Compute，套餐按上一节定，机房选东京、新加坡等亚太节点，操作系统选 Ubuntu 24.04，同时建好防火墙组（只放行 22、80、443）。
2. **装 Docker**。SSH 上去装 Docker；或者装 1Panel 这类面板用图形界面管理容器，对新手明显更友好。
3. **写 Compose 文件**，n8n 和 Caddy 两个容器一起起：

yaml
services:
  n8n:
    image: n8nio/n8n
    restart: always
    environment:
      - N8N_HOST=n8n.example.com
      - WEBHOOK_URL=https://n8n.example.com/
      - GENERIC_TIMEZONE=Asia/Shanghai
    volumes:
      - n8n_data:/home/node/.n8n
    expose:
      - '5678'
  caddy:
    image: caddy:latest
    restart: always
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
volumes:
  n8n_data:
  caddy_data:


Caddyfile 只要三行：


n8n.example.com {
    reverse_proxy n8n:5678
}


4. **解析域名**。把域名的 A 记录指向实例公网 IP，Caddy 启动后会自动申请 Let's Encrypt 证书，HTTPS 不用手动配。
5. **初始化**。浏览器打开域名，第一次会要求创建管理员账号，这一步别跳过，它是编辑器的登录凭证。

环境变量记住三个就够：`N8N_HOST` 填你的域名，`WEBHOOK_URL` 填带协议头的完整地址（webhook 回调全靠它），`GENERIC_TIMEZONE` 设成 `Asia/Shanghai` 让定时任务走对时区。

> 挂载数据卷比环境变量重要：`n8n_data:/home/node/.n8n` 必须写上。没挂载的话 n8n 照样能跑，但你下次升级镜像时，所有工作流和数据会被清空，且无法补救。首次部署忘了挂载、事后才补的，n8n 会直接初始化。

## 11 万台裸奔的 n8n：防火墙应该进你的选购清单

Censys 在 2026 年 10 月 14 日发布的年度互联网状态报告里给出一个扎眼数字：n8n 是公网上暴露最多的 AI 产品，113,926 台主机可被直接访问；可识别版本的实例里，76% 处于至少一个近期高危漏洞的影响范围内。美国、中国、德国三地占了暴露主机的一半以上。报告还点名了一个常见误区——68% 被暴露的 HTTP 服务开在非标准端口上，把 5678 改成 5679 躲不过全网扫描器，只是给自己添堵。

这直接影响 n8n 服务器的选择标准：防火墙能力应该写进你的选购清单。部署完成后花十分钟做完这几件事：

1. 在 Vultr 控制台建防火墙组，只放行 22、80、443，5678 对公网关闭；
2. 编辑器入口交给 Caddy 的 HTTPS 反向代理，全站强制加密；
3. 环境变量加 `N8N_BLOCK_ENV_ACCESS_IN_NODE=true`，阻止 Code 节点读取进程环境变量里的 API 密钥；
4. 升级 n8n 前先打快照，平时保持版本更新——76% 的漏洞率说明，最大的风险是过期版本。

## 常见问题

**1GB 内存能跑 n8n 吗？**
能启动，但不建议。2GB 是各方一致的底线，1GB 机器跑批量任务或带 AI 节点的工作流很容易内存溢出，Vultr 的 1GB 档留作部署演练更合适。

**Vultr 有 n8n 一键模板吗？**
目前没有专门的 n8n 应用模板，这是 Hostinger 这类商家的功能点。不过 n8n 的 Docker 镜像环境适应性很高，按上面的 Compose 文件走是十分钟级别的活。实在介意这一步，就选带模板的商家。

**需要备案吗？机房选哪里？**
Vultr 机房全部在海外，不需要备案。国内访问选东京、首尔、新加坡等亚太节点；如果你的工作流主要回调国内服务，开通后先实测一下延迟，比看任何评测都准。

**数据怎么备份？**
两层互为备份：Vultr 自动备份（加收实例价格的 20%）负责整机；工作流层面定期用 n8n 自带的导出功能存 JSON，两边互不依赖。

## 最后怎么选

按情况对号入座：

- 预算敏感、工作流以收发消息和 API 拼接为主 → Regular 2GB，每月 10 美元封顶；
- 生产环境、工作流带 AI 节点或数据量不小 → High Performance 4GB，每月 24 美元；
- 还在观望 → 开一台按小时计费的实例试跑一两天，总成本不到 1 美元，结论自然就有。

下单前确认两件事：机房位置（亚太节点对国内访问更友好），以及当期的新用户赠金活动——这个经常调整，以结账页显示为准。

[👉 开通你的 Vultr，部署自己的 n8n](https://bit.ly/VulTR)
