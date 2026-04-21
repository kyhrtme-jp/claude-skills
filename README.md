# claude-skills

今すぐ使える Claude Code 用スキル集。

## 含まれるスキル

### 📮 daily-report
セッションを締めくくる発言をすると、その日の作業内容を絵文字マーカー形式で整理し、指定した Google Chat スペースに自動送信してくれるスキル。

---

## セットアップ

### Step 1: Google Chat スペース / Webhook を用意

1. Google Chat で **個人専用のスペース**を作成（自分しか居ないスペースで OK）
   - 例: 「📔 yourname の作業ログ」
2. スペースを開いて、スペース名の横の **▼** → **アプリと連携** → **Webhook を追加**
3. 名前: `Claude Daily Report`、アイコン画像: 任意
4. 「保存」を押すと **Webhook URL** が表示される → **コピーして控える**

URL の形式:
```
https://chat.googleapis.com/v1/spaces/XXXXX/messages?key=YYYYY&token=ZZZZZ
```

### Step 2: リポジトリを clone

**GitHub アカウントは不要です。** ホームディレクトリ配下などお好きな場所に、以下のコマンドで clone できます:

```bash
git clone https://github.com/kyhrtme-jp/claude-skills.git ~/claude-skills
```

> ℹ️ `git` コマンドが入っていない場合は、ターミナルで `git --version` を実行して確認してください。未インストールなら macOS は Xcode Command Line Tools、Windows は [Git for Windows](https://gitforwindows.org/) を入れてください。

### Step 3: skill を Claude Code のスキルディレクトリへ配置

Claude Code はユーザーのスキルを `~/.claude/skills/` から読み込みます。

#### macOS / Linux

シンボリックリンクで配置するのが楽（将来の更新が `git pull` だけで済む）:

```bash
mkdir -p ~/.claude/skills
ln -s ~/claude-skills/skills/daily-report ~/.claude/skills/daily-report
```

コピーでも可:
```bash
cp -r ~/claude-skills/skills/daily-report ~/.claude/skills/
```

#### Windows（PowerShell）

Windows でのシンボリックリンク作成は管理者権限が必要なため、**コピー方式**を推奨します:

```powershell
New-Item -ItemType Directory -Path "$HOME\.claude\skills" -Force
Copy-Item -Path "$HOME\claude-skills\skills\daily-report" -Destination "$HOME\.claude\skills\daily-report" -Recurse -Force
```

> ℹ️ 将来スキルが更新された場合は、`git pull` した後に再度 `Copy-Item` を実行してください。

### Step 4: 環境変数を設定

#### macOS / Linux（zsh・bash）

シェルの設定ファイル（`~/.zshrc` or `~/.bashrc`）に以下を追記:

```bash
export CLAUDE_DAILY_REPORT_WEBHOOK="<Step 1 でコピーした Webhook URL>"
```

反映:
```bash
source ~/.zshrc   # or source ~/.bashrc
```

#### Windows（PowerShell）

PowerShell を**管理者権限なし**で開き、以下を実行（ユーザー環境変数として永続保存されます）:

```powershell
[Environment]::SetEnvironmentVariable("CLAUDE_DAILY_REPORT_WEBHOOK", "<Step 1 でコピーした Webhook URL>", "User")
```

設定後、**PowerShell ウィンドウを一度閉じて開き直す**と反映されます。

### Step 5: 動作確認

#### 5-1. 環境変数が設定されているか

**macOS / Linux:**
```bash
echo $CLAUDE_DAILY_REPORT_WEBHOOK
```

**Windows (PowerShell):**
```powershell
echo $env:CLAUDE_DAILY_REPORT_WEBHOOK
```

URL が出力されれば OK。

#### 5-2. Claude Code を起動してスキルを認識させる
Claude Code を一度終了して再起動する。

#### 5-3. テスト送信
Claude Code 内で以下のような発言をする:
```
今日はここまで、日報送って
```

Claude が skill を発火して、Google Chat スペースに日報が届けば成功。

---

## 使い方

**明示的な発言**
- 「今日はここまで」「区切りにする」「一旦完了」「日報送って」

**カジュアルな終わりの挨拶** でも発火:
- 「ありがとう、終わります」「お疲れさま」「また明日」「じゃあ今日はこれで」

Claude が当日のセッション内容を自動で整理して Chat に送信する。同じ内容が Claude のチャット上にも表示されるので、後から見返し可能。

## 日報のフォーマット

```
📅 YYYY-MM-DD 作業日報
🎯 テーマ1・テーマ2・テーマ3

🔧 機能改善
 • 項目1
 • 項目2

📖 ドキュメント
 • 項目1

🔒 セキュリティ
 • 項目1

⚠️ 残っている課題
 • 課題1

🔜 次のアクション
 • アクション1
```

カテゴリ絵文字は 🔧/🐛/🎨/📖/📝/🔒/⚙️/🧪/📊/🚀 など自動で選ばれる。

---

## トラブルシューティング

### 「環境変数が未設定です」と言われる

**macOS / Linux:**
- `~/.zshrc` に `export CLAUDE_DAILY_REPORT_WEBHOOK="..."` を追記したか？
- `source ~/.zshrc` で反映したか？
- `echo $CLAUDE_DAILY_REPORT_WEBHOOK` で値が出るか？

**Windows:**
- PowerShell で `[Environment]::SetEnvironmentVariable(...)` を実行したか？
- **PowerShell を一度閉じて開き直したか？**（現行ウィンドウには反映されない）
- `echo $env:CLAUDE_DAILY_REPORT_WEBHOOK` で値が出るか？

共通: Claude Code を再起動したか？

### Webhook URL が漏洩した疑い
1. Google Chat スペースで、該当 Webhook を **再生成** or **削除して作り直し**
2. 環境変数を新しい URL で更新
3. 反映: `source ~/.zshrc`（macOS/Linux）または PowerShell 再起動（Windows）

### skill が発火しない
- Claude Code 再起動してみる
- `~/.claude/skills/daily-report/SKILL.md` が存在するか確認
- 発言がカジュアル過ぎる or 作業途中と判断された可能性。「日報送って」と明示的に頼めば発火する

### 複数スペースに同時送信したい
現状は 1URL のみ対応。

---

## 更新方法

新バージョンが push されたら:

**macOS / Linux:**
```bash
cd ~/claude-skills && git pull
```
シンボリックリンクで配置していれば自動で反映。コピー配置の場合は Step 3 のコピーコマンドを再実行してください。

**Windows (PowerShell):**
```powershell
cd $HOME\claude-skills
git pull
Copy-Item -Path "$HOME\claude-skills\skills\daily-report" -Destination "$HOME\.claude\skills\daily-report" -Recurse -Force
```

---

## サポート / フィードバック

このリポジトリは配布目的で公開しており、**Issues / Pull Request は受け付けていません**。利用に関するご質問は配布元までお問い合わせください。

## ライセンス

[MIT License](LICENSE)
