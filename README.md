name: Auto Update Worker

on:
  push:
    branches:
      - main
  schedule:
    - cron: "0 1 * * *" # 每天凌晨1点运行
  workflow_dispatch:
    inputs:
      force_update:
        description: '是否强制更新（忽略版本检查）'
        required: false
        default: 'false'

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: 检出仓库
        uses: actions/checkout@v4

      - name: 设置环境
        run: |
          echo "REPO_URL=https://api.github.com/repos/bia-pain-bache/BPB-Worker-Panel/releases" >> $GITHUB_ENV
          echo "TARGET_FILE=worker.zip" >> $GITHUB_ENV

      - name: 检查并更新 Worker
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # 使用 GitHub Token 认证
        run: |
          # 日志函数
          log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"; }

          log "开始检查更新..."

          # 获取本地版本
          LOCAL_VERSION=$(cat version.txt 2>/dev/null || echo "")
          log "本地版本: ${LOCAL_VERSION:-无}"

          # 获取最新 Release
          log "获取最新 Release 信息..."
          RESPONSE=$(curl -s --retry 3 -H "Authorization: token $GITHUB_TOKEN" -H "Accept: application/vnd.github.v3+json" "$REPO_URL")
          if [ $? -ne 0 ]; then
            log "ERROR: 无法访问 GitHub API"
            exit 1
          fi

          TAG_NAME=$(echo "$RESPONSE" | jq -r '.[0].tag_name')
          DOWNLOAD_URL=$(echo "$RESPONSE" | jq -r '.[0].assets[] | select(.name == "'"$TARGET_FILE"'") | .browser_download_url')

          if [ -z "$DOWNLOAD_URL" ] || [ "$DOWNLOAD_URL" == "null" ]; then
            log "ERROR: 未找到 $TARGET_FILE"
            exit 1
          fi
          log "最新版本: $TAG_NAME"

          # 判断是否需要更新
          FORCE_UPDATE=${{ github.event.inputs.force_update || 'false' }}
          if [ "$LOCAL_VERSION" = "$TAG_NAME" ] && [ "$FORCE_UPDATE" != "true" ]; then
            log "已是最新版本，无需更新"
            exit 0
          fi

          # 下载并更新
          log "下载 $TARGET_FILE..."
          wget -q -O "$TARGET_FILE" "$DOWNLOAD_URL"
          log "解压 $TARGET_FILE..."
          unzip -o "$TARGET_FILE"
          rm "$TARGET_FILE"
          echo "$TAG_NAME" > version.txt
          log "更新完成，新版本: $TAG_NAME"

      - name: 提交更改
        if: success() # 仅在更新成功时提交
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "🔄 自动同步 Worker 版本: ${{ steps.check_update.outputs.tag_name || '未知' }}"
          commit_author: "github-actions[bot] <github-actions[bot]@users.noreply.github.com>"

          最新BPB面板搭建
一、准备工作
1. GitHub 账号：通过 Github Action 自动同步最新 BPB 源代码。
2. Cloudflare 账号：用于部署 BPB Panel 项目。
3. 域名：建议使用域名（解决 Cloudflare Pages 自带域名被墙的问题）。
在仓库根目录下创建 .github/workflows 文件夹，并在其中创建 update-worker.yml 文件。

直接复制：.github/workflows/update-worker.yml
复制下面的代码：

name: Auto Update Worker

on:
  push:
    branches:
      - main
  schedule:
    - cron: "0 1 * * *" # 每天凌晨1点自动运行
  workflow_dispatch:     # 支持手动运行

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: 初始化仓库
        uses: actions/checkout@v4

      - name: 获取当前本地版本
        id: get_local_version
        run: |
          echo -e "\033[34m[获取本地版本]\033[0m"
          if [ -f version.txt ]; then
            LOCAL_VERSION=$(cat version.txt)
            echo "当前本地版本: $LOCAL_VERSION"
          else
            echo "首次同步，没有本地版本。"
            LOCAL_VERSION=""
          fi
          echo "LOCAL_VERSION=$LOCAL_VERSION" >> $GITHUB_ENV

      - name: 获取最新 Release 信息
        id: get_release
        run: |
          echo -e "\033[34m[获取最新 Release]\033[0m"
          API_URL="https://api.github.com/repos/bia-pain-bache/BPB-Worker-Panel/releases"
          RESPONSE=$(curl -s "$API_URL")
          LATEST_RELEASE=$(echo "$RESPONSE" | jq -r '.[0]')
          TAG_NAME=$(echo "$LATEST_RELEASE" | jq -r '.tag_name')
          DOWNLOAD_URL=$(echo "$LATEST_RELEASE" | jq -r '.assets[] | select(.name == "worker.zip") | .browser_download_url')

          if [ -z "$DOWNLOAD_URL" ] || [ "$DOWNLOAD_URL" == "null" ]; then
            echo -e "\033[31m未找到 worker.zip，退出！\033[0m"
            exit 1
          fi

          echo "最新版本号: $TAG_NAME"
          echo "DOWNLOAD_URL=$DOWNLOAD_URL" >> $GITHUB_ENV
          echo "TAG_NAME=$TAG_NAME" >> $GITHUB_ENV

      - name: 判断是否需要更新
        id: check_update
        run: |
          echo -e "\033[34m[判断是否需要更新]\033[0m"
          if [ "$LOCAL_VERSION" = "$TAG_NAME" ]; then
            echo -e "\033[32m已经是最新版本，无需更新。\033[0m"
            echo "UPDATE_NEEDED=false" >> $GITHUB_ENV
          else
            echo -e "\033[33m发现新版本，需要更新！\033[0m"
            echo "UPDATE_NEEDED=true" >> $GITHUB_ENV
          fi

      - name: 如果需要，清理旧文件并下载新版本
        if: env.UPDATE_NEEDED == 'true'
        run: |
          echo -e "\033[34m[清理旧文件]\033[0m"
          rm -rf ./*
          echo -e "\033[34m[下载最新 worker.zip]\033[0m"
          wget -O worker.zip "$DOWNLOAD_URL"
          echo -e "\033[34m[解压 worker.zip]\033[0m"
          unzip worker.zip
          echo -e "\033[34m[删除 worker.zip]\033[0m"
          rm worker.zip
          echo -e "\033[34m[记录新版本号]\033[0m"
          echo "$TAG_NAME" > version.txt

      - name: 提交更改
        if: env.UPDATE_NEEDED == 'true'
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "🔄 自动同步最新 Worker 版本：${{ env.TAG_NAME }}"
          commit_author: "github-actions[bot] <github-actions[bot]@users.noreply.github.com>"
          push_options: --force

三、Cloudflare 部署

1在 Cloudflare 控制台中进入 Workers 和 Pages，选择 Pages 部署。

2. 绑定自定义域名（可选）

3. 设置变量

UUID：使用 UUID 生成器 随机生成一个新的 UUID。
PROXY_IP：填写代理 IP 地址，可从 随机代理 IP 站点 获取，或使用优选域名（例如 cdn-b100.xn--b6gac.eu.org）。
TR_PASS：填写一个复杂字符串，作为密码。
创建 KV:点击左侧存储和数据库，再选择 KV,然后创建一个新的 KV 命名空间

注:变量名称只能填写“kv”(小写)

打开浏览器输入:https://[自定义域名]或者你的项目地址,后面加上/panel,检查是否能正常访问BPB面板

Proxy IPs / Domains 获取地址：点击访问	代理 IP/域名	141.147.156.68、cdn-xx-b6gac.acu.org
Clean IPs / Domains	清洁 IP/域名	104.17.212.246、104.19.19.167

https://wandou.eu.org/archives/171

