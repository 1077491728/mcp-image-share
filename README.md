# mcp-image-share

基于 `gpt-image-2` 的 MCP Server 图片生成分享工具，支持文生图、图编辑、批量编辑、多图融合。适配 Claude Code / Codex / Cursor 等任意 MCP 客户端。

---

## 功能

| Tool | 说明 | 典型耗时 |
|---|---|---|
| `image_generate` | 文生图 | ~30-45s |
| `image_edit` | 单图编辑（支持 alpha mask） | ~30-45s |
| `image_batch_edit` | N 进 N 出，每张同指令独立处理，5 并发 | N×30s |
| `image_multi_reference` | 2-10 张参考图融合输出 1 张新图 | ~30-100s |
| `server_info` | 路由规则 / size 矩阵 / 安全约束诊断 | — |

---

## 实际输出像素（实测验证）

此端点所有请求统一输出 ~1.57MP，宽高比保持正确：

| 请求 Size | 实际输出 | 像素 |
|-----------|----------|------|
| 1024x1024 | 1254x1254 | 1.57MP |
| 1280x720 | 1672x941 | 1.57MP |
| 720x1280 | 941x1672 | 1.57MP |
| 1024x1536 | 1024x1536 | 1.57MP |
| 1536x1024 | 1536x1024 | 1.57MP |

---

## 安装

```bash
git clone https://github.com/1077491728/mcp-image-share.git
cd mcp-image-share
pip install -e .
```

---

## 配置

### 一键安装（推荐）

```bash
python install.py
```

脚本自动安装依赖、交互配置 key、写入 Claude Code 和 Codex CLI 配置。

选项：
```bash
python install.py --no-codex          # 只写 Claude Code
python install.py --no-claude         # 只写 Codex
python install.py --mirror tsinghua   # pip 走清华镜像
python install.py --yes               # 非交互（从 env 读）
```

### Claude Code 手动配置

在项目目录下创建 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "mcp-image-share": {
      "command": "python",
      "args": ["-m", "server"],
      "cwd": "<clone 路径>",
      "env": {
        "MICU_API_KEY": "sk-your-key",
        "MICU_BASEURL": "https://new.misscuai.help",
        "MICU_SAVE_DIR": "~/Pictures/micu-out"
      }
    }
  }
}
```

### Codex CLI 手动配置

编辑 `~/.codex/config.toml`：

```toml
[mcp_servers.mcp-image-share]
command = "python"
args = ["<clone 路径>/server.py"]
env = { MICU_API_KEY = "sk-your-key", MICU_BASEURL = "https://new.misscuai.help", MICU_SAVE_DIR = "~/Pictures/micu-out" }
```

配置完重启客户端即可。

---

## 环境变量

| 变量 | 必填 | 默认 | 说明 |
|---|---|---|---|
| `MICU_API_KEY` | ✅ | — | API Key |
| `MICU_BASEURL` | ❌ | `https://new.misscuai.help` | API 端点 |
| `MICU_MODEL` | ❌ | `gpt-image-2` | 默认模型 |
| `MICU_SAVE_DIR` | ❌ | `~/Pictures/micu-out` | 输出目录 |

---

## 用法

直接对 LLM 说：

```
画一张 1024x1024 的赛博朋克猫咪
把 ~/Pictures/cat.png 的背景换成海边
给这几张产品图统一加水印
结合这 3 张参考图，画一张同风格的新场景
```

---

## 安全约束

- `base_url` 锁定在启动期 env，运行期不接受参数
- 输出目录强制在 `MICU_SAVE_DIR_ROOT` 之下
- 输入图按 magic bytes 校验为 PNG/JPEG/WebP/GIF
- 单图 ≤ 4MB，多图总和 ≤ 8MB
- `n ∈ [1, 10]`，防 burn quota
- 响应 ≤ 25MB，超过中断不落盘

---

## 可用模型

- `gpt-image-1`
- `gpt-image-2`（默认）
- `grok-imagine-image-lite`

---

## 许可

MIT
