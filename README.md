# mc服务器搭建：从选配置到装Mod，新手也能和朋友开一个稳定联机的私服

搜「mc服务器搭建」的人，真正卡住的通常是三件事：机器买多大、服务端选哪个、朋友怎么连进来。先给结论——2到5人玩纯净服，一台4核4G的云服务器就够，参考活动价最低38元/年，5个人AA每人不到8元；要装30个以上Mod，直接上4核8G；大型整合包或20人服，8核16G起步。

想省事的话，路径可以压缩成三步：买一台预装Docker的云服务器、一条命令拉起Paper服务端、控制台加白名单。全程不用手动装Java。

> **选购速览**
> - 2-5人，原版或Paper插件服：4核4G，参考价38-99元/年
> - 6-10人，或3-5人玩30-80个Mod的整合包：4核8G，参考价约450元/年
> - 11-20人，或80+ Mod的大型整合包：8核16G，参考价约1800元/年
> - 👉 [查看腾讯云轻量服务器当前活动价](https://bit.ly/yunhuodong)

## 先定三件事，再谈搭建

搭建之前有三个决定会影响后面所有步骤：Java版还是基岩版、用什么服务端、预算按年还是按月。这三件事定错了，配置买完还得退。

**Java版 vs 基岩版**

| 版本 | 客户端平台 | 服务端类型 | Mod支持 | 跨平台联机 |
| --- | --- | --- | --- | --- |
| Java版 | PC（Windows/Mac/Linux） | 多种可选 | 丰富 | 不支持 |
| 基岩版 | 手机/主机/Win10 | Bedrock Dedicated Server | 有限 | 全平台 |

Java版是PC玩家的主战场，Mod生态最全；基岩版覆盖手机、主机和Win10，能跨平台联机，但Mod支持有限。朋友里有手机玩家的话，要么选基岩版，要么接受Java版只能PC进。下文以Java版为主。

**服务端类型**

| 服务端 | 特点 | Mod兼容性 | 性能 | 推荐度 |
| --- | --- | --- | --- | --- |
| Vanilla（原版） | 官方服务端 | 仅原版 | 一般 | 不推荐 |
| Paper | 优化版，基于Spigot | Bukkit/Spigot插件 | 高（比原版提升30-50%） | 强烈推荐 |
| Purpur | 基于Paper，自定义更多 | Bukkit/Spigot插件 | 高 | 推荐 |
| Forge | Mod服务端标准 | Forge Mod | 中等 | Mod必选 |
| Fabric | 轻量Mod框架 | Fabric Mod | 较高 | 轻量Mod推荐 |

对应关系很简单：纯净服或插件服选Paper，要装Forge Mod选Forge，轻量Mod选Fabric。原版服务端性能一般，除非你有特殊理由，否则不建议。

## 配置怎么选：先算内存，再看CPU

MC服务器最先吃满的通常是内存，CPU反而没那么紧张。一台2核机器只要内存够，开4-5人小服没问题；内存不够时，核心再多也会卡在区块加载上。

内存的粗略算法：

- 服务端基础占用：1-1.5GB
- 每个在线玩家：200-400MB
- 每个Mod：50-200MB
- 世界探索范围越大，占用越高

按这个公式，5个玩家加20个Mod大约需要 1.5 + 5×0.3 + 20×0.1 ≈ 5GB，所以4核8G的机器给MC分5-6G是合理档位。这也是很多教程只写「4G够用」却没说清的前提——够用的是纯净服，不是整合包。

### 腾讯云轻量应用服务器：按开服场景选档位

腾讯云轻量应用服务器可以选Docker CE应用镜像购买，Docker环境预装好，买完直接跑MC服务端。面向开服的三个主流档位如下（价格为教程标注的参考价，活动价通常限新用户首单，实际以下单页为准）：

| 配置档位 | 适用场景 | 建议分配给MC | 参考年费 | 购买入口 |
| --- | --- | --- | --- | --- |
| 4核4G | 2-5人原版/Paper纯净服 | 2-3G | 38-99元 | [查看4核4G档活动价](https://bit.ly/yunhuodong) |
| 4核8G | 6-10人纯净服；3-5人中型整合包（30-80 Mod） | 5-6G | 约450元 | [查看4核8G档活动价](https://bit.ly/yunhuodong) |
| 8核16G | 11-20人纯净服；80+ Mod大型整合包 | 10-12G | 约1800元 | [查看8核16G档活动价](https://bit.ly/yunhuodong) |

有两个容易忽略的点：38元/年是最低特惠档，不是所有配置都这个价；首单特惠和续费价通常不一样，打算长期开服的话，下单前先看清计费周期。只玩一个假期，按月买更灵活。👉 [对比各档位当前价格与计费方式](https://bit.ly/yunhuodong)

## 部署：Docker一条命令，还是手动装

新手直接用Docker，省掉装Java和配环境变量的步骤。SSH连上服务器后执行：

bash
docker run -d --name mc-server \
  -p 25565:25565 \
  -e EULA=TRUE \
  -e TYPE=PAPER \
  -e VERSION=1.21.4 \
  -e MEMORY=2G \
  -e MOTD="我们的MC私服" \
  -e MAX_PLAYERS=10 \
  -e DIFFICULTY=normal \
  -e VIEW_DISTANCE=8 \
  -v mc-data:/data \
  --restart=always \
  itzg/minecraft-server


参数里值得改的几个：`TYPE` 选 PAPER、FORGE 或 FABRIC；`VERSION` 填你的Mod要求的版本；`MEMORY` 按机器配置分配，4G机器分2G，8G机器分5-6G；`VIEW_DISTANCE` 设8-10，视距越低越省性能。等2-3分钟镜像下载完，客户端输入「你的IP:25565」就能进。

习惯手动控制的，可以选Ubuntu镜像自己装。Java版本要按MC版本对：

| MC版本 | 需要Java版本 |
| --- | --- |
| 1.16及以下 | Java 8 |
| 1.17-1.20.4 | Java 17 |
| 1.20.5及以上 | Java 21 |
| 26.1快照起 | Java 25 |

手动启动建议带GC优化参数，并用screen把进程放后台，断开SSH也不会停服：

bash
java -Xms2G -Xmx2G \
  -XX:+UseG1GC -XX:+ParallelRefProcEnabled \
  -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions \
  -XX:+DisableExplicitGC -XX:G1NewSizePercent=30 \
  -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M \
  -jar server.jar nogui


用Docker部署的话，Java版本由镜像自动匹配，这是它比手动安装省心的地方。👉 [用预装Docker的镜像开通服务器](https://bit.ly/yunhuodong)

## Mod怎么装，装完怎么不卡

Mod服能不能连上，取决于服务端框架和客户端Mod版本是否严格一致。Forge、Fabric、Paper各自的Mod/插件不通用，装错框架，客户端会直接提示版本不兼容。

Forge服加Mod就两步，把jar传进容器再重启：

bash
docker cp your-mod.jar mc-forge:/data/mods/
docker restart mc-forge


即使是Mod服，也建议加几个性能优化Mod：

| Mod名称 | 适用框架 | 作用 |
| --- | --- | --- |
| Lithium | Fabric | 优化游戏逻辑 |
| Starlight | Fabric/Forge | 重写光照引擎 |
| FerriteCore | Fabric/Forge | 减少30-50%内存占用 |
| Chunky | Bukkit/Fabric | 预生成地图区块，消除探索卡顿 |

整合包方面，All The Mods 10（400+ Mod）建议8核16G；RLCraft、Better Minecraft这类200个Mod左右的，4核8G能跑；FTB Presents Direwolf20、Create: Above and Beyond大约100-200个Mod，同样落在4核8G档。

## 开服之后：白名单、备份、定时重启

服务器开起来只是开始，这三件事决定你能不能睡得安稳。

白名单是私服的第一道门，控制台执行 `whitelist add 好友ID`，陌生人就进不来。别嫌麻烦，被拆一次家就晚了。

自动备份：MC存档损坏不是小概率事件。用脚本每天凌晨把world目录拷出来，只保留最近7天：

bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M)
BACKUP_DIR="/backup/minecraft"
mkdir -p $BACKUP_DIR
docker exec mc-server rcon-cli save-all
sleep 5
docker cp mc-server:/data/world $BACKUP_DIR/world_$DATE
find $BACKUP_DIR -name "world_*" -mtime +7 -exec rm -rf {};


挂进crontab：`0 4 * * * /opt/mc-backup.sh`。顺手再加一条 `0 4 * * * docker restart mc-server`，每天凌晨重启一次，释放长时间运行积累的内存。

常用管理命令备查：

| 命令 | 作用 |
| --- | --- |
| `op 玩家名` | 给予管理员权限 |
| `whitelist add 玩家名` | 添加白名单 |
| `ban 玩家名` | 封禁玩家 |
| `gamerule keepInventory true` | 死亡不掉落 |
| `save-all` | 手动保存世界 |
| `stop` | 安全关闭服务器 |

## 常见问题

**连接不上服务器？**
先查防火墙是否放行25565端口（TCP），再确认MC客户端版本和服务端版本一致。

**需要买多大内存？**
按「基础1-1.5GB + 每人200-400MB + 每个Mod 50-200MB」估算，再留1GB余量，宁多勿少。

**启动报Java版本错误？**
1.16及以下用Java 8，1.17-1.20.4用Java 17，1.20.5及以上用Java 21。用Docker镜像会自动匹配，手动装容易在这里踩坑。

**手机能进Java服吗？**
不能。Java版和基岩版不互通，手机玩家需要单独开基岩版服务端。

**38元/年的机器能开Mod服吗？**
4核4G跑2-5人纯净服可以；30个Mod以上的整合包建议直接上4核8G，不然内存溢出会频繁崩服。

## 最后：按你的情况对号入座

两三个人、纯原版、只玩一个假期，选4核4G，参考价38-99元/年；5-10人长期服或少量插件，选4核8G，约450元/年，内存分5-6G；要开大型整合包或20人社区服，8核16G，约1800元/年，别在内存上省钱。

完全不想花钱也有路：自家电脑加内网穿透能开服，但电脑不能关机，网络波动也会原样传给所有人，适合临时联机，不适合长期服。

下单前确认两件事：活动价是否限新用户首单，以及你选的配置能不能覆盖Mod包的内存需求。👉 [查看腾讯云轻量服务器最新活动与配置](https://bit.ly/yunhuodong)
