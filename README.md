# YouTube 字幕归档（文本仓库）

目标：用上游 [youtube-caption-tool](https://github.com/qianlicaoemma-hub/youtube-caption-tool) 抽出长视频全文，**先完整归档**，读完后再挑点写公众号。

本 Git 仓库只跟踪**文本与笔记**（`transcripts/`、`videos/`、`articles/`、本 README）。  
上游抽取工具在本地的 `tool/` 目录运行，**不纳入本仓库**（见 `.gitignore`），避免嵌套 `.git`、`.venv`、密钥与输出缓存。

## 目录

```
videos/<日期-slug>/   # 每条视频的来源 URL 与笔记
transcripts/raw/      # 工具完整导出（优先归档）
transcripts/clean/    # 可读清洗稿（可选）
articles/             # 公众号成稿（后置，暂空）
tool/                 # 本地专用：上游工具克隆（被 gitignore，需自行 clone）
```

## 本地准备抽取工具（一次性）

在仓库根目录：

```bash
cd "/Users/ruiblue/Projects/学习工具/youtube-caption-tool"
git clone https://github.com/qianlicaoemma-hub/youtube-caption-tool.git tool
cd tool
# 需要 Python ≥3.11；本机可用 uv：
#   uv python install 3.12 && uv venv .venv --python 3.12
#   source .venv/bin/activate && uv pip install -r requirements.txt
cp .env.example .env
# 仅字幕试跑：把 PUBLIC_MODE=true；无字幕再配火山引擎四件套
```

若本机已有 `tool/`（含 `.venv`），可跳过 clone。

## 启动

```bash
cd "/Users/ruiblue/Projects/学习工具/youtube-caption-tool/tool"
source .venv/bin/activate
uvicorn app.main:app --host 127.0.0.1 --port 8000
```

浏览器：http://127.0.0.1:8000  
或双击 `tool/start.command`。

`PUBLIC_MODE=true` 时仅字幕、零成本；提交 API 时 `cookies_from_browser` 必须为空字符串。

## 用 API 抽字幕（已验证）

```bash
curl -s -X POST http://127.0.0.1:8000/api/jobs \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://www.youtube.com/watch?v=VIDEO_ID","mode":"captions","language":"auto","cookies_from_browser":""}'
```

完成后把 `tool/app/outputs/<job_id>/*.md` 复制到 `transcripts/raw/<slug>.md`，并更新 `videos/<slug>/NOTES.md`。

## 首次试跑

- URL：https://www.youtube.com/watch?v=V9eI-t3TApE
- raw：`transcripts/raw/2026-11-07-V9eI-t3TApE.md`

## 推送到 GitHub（私有库）

本机若尚未登录 `gh`：

```bash
brew install gh   # 如未安装
gh auth login
cd "/Users/ruiblue/Projects/学习工具/youtube-caption-tool"
gh repo create youtube-caption-archive --private --source=. --remote=origin --push
```

或网页新建空私有库后：

```bash
git remote add origin git@github.com:<你的用户名>/youtube-caption-archive.git
git push -u origin main
```
