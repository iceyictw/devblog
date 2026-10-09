---
title: Github Actions 實例：自架伺服器自動部署 Blog
date: 2026-09-25T00:01:27+08:00
draft: true
summary: "以自架伺服器部署 Devblog 為例，介紹如何設定 GitHub Actions Self-hosted Runner，並透過 Workflow 自動建置與部署網站。從 Workflow 的基本結構、Runner 的目錄管理，到環境變數與 Context 的差異，逐步說明 GitHub Actions 的運作方式。"
tags: ["learning", "linux"]
---

# 前言
最近有了在住處自架一個簡易 Homelab 的想法。
首先想到的是可以把這個 Devblog 從 Github Page 移動到自己的主機上，
同時在工作上也遇到一些可以利用 Github Actions 達到方便快速地更新服務的情境。
然而在筆者實際摸索時，發現整套 Github Actions 的系統相當有彈性，可以應付小型大型的測試 / 部署需求，但也因此我認為這不是一個容易入門的系統。
故我在此以我的 Devblog 自動化部署為例，一步步地說明設置的過程，提供一個學習的出發點。



# 設定 Self-hosted Runner
在你的 Github Repo 上，開啟 Setting > Actions > Runners，然後選擇 `Add new self-hosted runner`。
裡面會指示你在自己的 Runner 上下載 runner package，然後使用 `./config.sh` 設定。
設定時需要提供 repo url 跟驗證用的 token，讓 Github 允許這個 Runner 加入到 repo 中。

設定時，`config.sh` 會詢問是否要為該 runner 加入標籤，標籤可以讓 workflow 在執行時能夠篩選可以分配工作的 runner，
在這裡，我新增了一個 `development` 的標籤，用來表示這是部署測試環境用的主機。

{{<figure src="runner_config.png" align="center">}}

完成以上步驟後，就可以在 runners 列表看到自己的 runner。

{{<figure src="runners_list.png" align="center">}}

最後需要啟動 runner。Github 雖然指示你直接執行 `./run.sh`，但這種方式會佔據你的 shell process，因此更好的方式是把它設定成一個 systemd service。
runner package 提供 `./svc.sh` 可以快速地幫助你安裝 service 然後啟動。
```
$ sudo ./svc.sh install
$ sudo ./svc.sh start
$ sudo ./svc.sh status
```

可以回到 Runners 頁面查看自己的 runner status 是否顯示上線。



# 撰寫 Workflow
在 `.github/workflows` 目錄中，每一個 `.yml / .yaml` 檔案就代表了一個 workflow。以下我們先釐清 workflow 中的層級概念：
- 一個 workflow 代表一套想要完整執行的目標，Github Actions 執行時就是以一個 workflow 為單位去完成。
- 一個 workflow 由一個或多個 jobs 組成。Github Actions 會以 job 為單位，派發到 runner 上，因此一個 workflow 中的 jobs 可能是由不同的 runners 協力完成的。jobs 之間可以定義依賴關係（哪個 job 必須在哪個 job 完成之後才能開始。）
- 一個 jobs 由多個 runs / actions 構成。run 是一個想要機器執行的 shell command process，actions 本質上也是一個 process，只是由 Github 上的其他 action 專案包裝好一個特定的行為。runner 會按照順序執行指定的 runs / actions。

接下來我們以上面部署 devblog 為例子：
```yaml
name: Deploy site to self-hosted development server

on:
  push:
    branches:
      - dev
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: development
  cancel-in-progress: false

jobs:
  build:
    runs-on: [self-hosted, development]
    steps:
      - uses: actions/checkout@v7
        with:
          submodules: recursive
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run build
      - run: |
          mkdir -p /srv/devblog/release/${{ github.sha }}/devblog
          rsync -a --delete public/ /srv/devblog/release/${{ github.sha }}/devblog
          rm -f /srv/devblog/current
          ln -s /srv/devblog/release/${{ github.sha }} /srv/devblog/current
```

- `on`: workflow 的觸發條件。例如可以設定在某個 branch 有 push 時、某個 branch 的 PR 被創建時、符合特定 Regex 的 Tag 被建立時，或是允許管理者手動觸發。
- `permissions`: 允許 runner 什麼樣的權限。有些 workflow 只是做 testing 時，通常只需要 read 權限就夠了，如果是想要達成自動 commitment 的情境，可能就需要允許對 Repo 進行寫入。
- `jobs`: 在這裡面定義你的每一個 job。上面的範例中，只有一個 job-id 為 `build` 的 job。
- `jobs.<job_id>.runs-on`: 這個欄位可以指定 job 要分配給什麼樣的 runner。上面使用了 `self-hosted` 代表只運行在自架的 runner 上，然後 runner 還必須有 `development` 這個標籤。這個標籤是我用來代表開發伺服器的標籤。假設我的專案今天想要分別有測試環境跟正式部署環境，那麼我就可以有兩台不同的 runner，並且給他們不同的標籤來區分。

接下來，我們可以看到 `steps` 裡面有五個 `uses / run` 子項目，runner 會依序執行這五個 action / shell command。讀者可以從字面意思了解執行內容，白話來說就是以下的步驟：
1. 用 `actions/checkout@v7` clone 出 repo 中的內容。
1. 用 `actions/setup-node@v4` 建立 nodejs 環境。
1. 安裝專案的 node packages。
1. 執行專案的 build 流程。
1. 把生出來的靜態檔案整包丟到 `/srv/devblog/release` 裡面交給 nginx 處理。



# 瞭解目錄結構
我在撰寫 workflow 時遇到的一個小難題就是搞不懂 runner 如何管理目錄結構，因此在這邊簡單說明整理。

假設 runner 的路徑為 `~/my-runner`，repo 名稱為 my-repo，那麼你的每一個 step 預設的 working directory 都是 `~/my-runner/_work/my-repo/my-repo/`

讀者可能會疑惑「為什麼路徑裡面會有兩層 `my-repo` 目錄？」第一層存在的目的是因為一個 runner 可能可以接受來自不同 repo 的 job。例如今天 repo-A 跟 repo-B 都登錄了這個 runner，那麼 `~/my-runner/_work/repo-A/` 就會用來放置執行 repo-A 的 job 產生的檔案，而 `~/my-runner/_work/repo-B/` 亦同。第二層則是因為 job 可能可以 checkout 多個 repo，例如 `~/my-runner/_work/my-repo/my-repo/` 跟 `~/my-runner/_work/my-repo/another-repo/`。

如果想要修改 working directory，可以透過 `jobs.<job_id>.steps[*].working-directory` 來設定該特定步驟的 working directory，或是透過 `jobs.<job_id>.defaults.run.working-directory` 來為整個 job 修改預設的 working directory。



# 環境變數與 context
Github 有兩種方式可以傳遞資訊給 runner。

首先先介紹環境變數 (environment variable)。這裡的環境變數指的就是 runner process 會看到的作業系統環境變數。我們可以給一個 step、一個 job 或是整個 workflow 指定環境變數。看一下以下的例子：
```yaml
name: Greeting on variable day

on:
  workflow_dispatch

env:
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
        env:
          First_Name: Mona
```
這個 workflow 在三個不同的層級都設定了一些環境變數，這些變數會傳遞給 runner，並套用在 runner 執行 step 啟動的 process 上，因此該 echo 指令將會輸出：
```
Hello Mona. Today is Monday!
```

另一方面，上下文變數 (context variable) 則是 Github Action 在 workflow 被執行時，Github 自動帶入的數值。舉例來說，我上面 devblog 的 workflow 有這麼一行：
```
mkdir -p /srv/devblog/release/${{ github.sha }}/devblog
```
其中的 `${{ github.sha }}` 就是 `github` 這個 context 的 `sha` 屬性，代表觸發 workflow 的 commit 之雜湊值。Context variables 可以取得 workflow 相關的各種資訊，還有 repository variable 等等。可以參考 [Github Context reference](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts) 了解有哪些 context 可以使用。

Context variable 跟 environment variable 不同之處在於，context variable 在 workflow 開始時會由 Github 決定好數值並填入，runner 並不知道 context variable 的存在。以上面的指令來說，runner 的 shell 實際上收到的指令是 `mkdir -p /srv/devblog/release/abc123.../devblog`，因此 runner 無法取得有關 context 的資訊。


# Reference
https://www.reddit.com/r/devops/comments/1favu4a/why_is_github_actions_doubling_my_working/
https://docs.github.com/en/actions/reference/workflows-and-actions/variables
https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#github-context