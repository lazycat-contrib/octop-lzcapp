# octop-lzcapp

[Octop](https://github.com/TencentCloud/Octop)（腾讯云开源的自托管 AI 助手）的懒猫微服打包：**只发喵喵商店，镜像模式**。

- **镜像**：`ghcr.io/tencentcloud/octop`，先用 tag 路径发的 1.0.2b5。`delivery.mode: lazycat` 会把镜像转存到懒猫镜像源并改写 manifest 里的 image（缓存与加速都靠它）。
- **版本映射**：上游 beta tag 不合法（`1.0.2b5`），按 beta 家族跟踪并用 `version_regex`/`version_template` 映射成合法 SemVer `1.0.2-b5`（与 `package.yml` 一致）。上游出 `1.0.2` 正式版时，把 `tag_regex` 换成 `^[0-9]+\.[0-9]+\.[0-9]+$` 即可切回稳定线。
- **路由**：`/` → `octop:8088`。入口保留微服账号校验（自托管助手不必对外公开）；要让 IM 回调或外部 Agent API 直连，把 `/` 加进 `application.public_path`。
- **环境**（照上游 Docker 文档）：
  - `HOME=/data` 是硬要求，`~/.octop` 才会落到数据目录；
  - `OCTOP_BIND_HOST=0.0.0.0`、`OCTOP_PORT=8088`；
  - `OCTOP_DEFAULT_PASSWORD` 来自安装向导；留空或不合策略（≥8 位含字母数字）时应用会自动改用随机密码写进 `credential.txt`，**不会因此起不来**；
  - 模型 Key 也可走 `OPENAI_API_KEY` / `DASHSCOPE_API_KEY` 等环境变量（不配就在「管理 → 模型」里加）。
- **持久化**：`/lzcapp/var/octop:/data/.octop`（SQLite、credential.txt、凭据、Agent 工作区）。
- **健康检查**：`curl -fsS http://127.0.0.1:8088/api/health`（镜像里自带 curl）。
- **`user: root`**：镜像是 python:3.12-slim、默认 root，而 `/lzcapp/var/octop` 由平台以 root 创建。
- **图标**：直接用上游 `dashboard/public/pwa-512.png`（512×512，压到 163KiB）。

## 商店状态

- **喵喵商店**：`cloud.lazycat.app.octop` **1.0.2-b5 已上架**（appId 518 / versionId 2224）。镜像已转存为 `registry.lazycat.cloud/czyt/tencentcloud/octop:02980826266165ac` 并写回清单；发布运行 [37088316371](https://github.com/lazycat-contrib/octop-lzcapp/actions/runs/37088316371)。之后每 6 小时探一次上游镜像，新 beta（1.0.2b6…）会自动走同一条链路。
- **官方商店**：`stores.official.enabled: false` —— 暂不上架；需要时补 PC/手机截图与应用信息再打开。
- 注：容器内行为以镜像自带健康检查（`/api/health`）为准；本仓库未在真机安装验证过登录、建模与对话流程。
