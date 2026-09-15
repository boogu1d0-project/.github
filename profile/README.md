# BOOGU项目代码协作指引

> 工具假设：已安装 Git，使用 HTTPS 访问 GitHub（不是 SSH）

---

## 一、创建 Personal Access Token (classic) 作为 HTTPS 密码

由于 GitHub 不再允许使用账号密码进行 Git 操作，而内网环境不允许以 SSH 协议访问 GitHub，我们需要创建一个 **Personal Access Token (classic)**，在 `git push` / `git pull` 时作为「密码」使用。

### 1.1 打开 GitHub 设置页面

1. 登录 GitHub（浏览器）。
2. 右上角头像 → `Settings`。
3. 左侧菜单最底部：`Developer settings`。
4. 左侧：`Personal access tokens` → 选择 **`Tokens (classic)`**。

### 1.2 创建 Token

1. 点击 **`Generate new token (classic)`**。
2. 填写：
   - **Note**：随便写一个说明，比如：`git https access for BOOGU`。
   - **Expiration**：建议选一个到期时间（比如 90 天），到期后再重新生成，嫌麻烦则选择 `No expiration`。
3. 勾选权限（最少权限原则）：
   - `repo`（包括 `repo:status`, `repo_deployment`, `public_repo`, `repo:invite`, `security_events`）
   - `read:org`
   - 一般不需要勾选 `admin` 开头的内容。

4. 页面底部 → 点击 **`Generate token`**。

### 1.3 保存 Token

GitHub 会显示一串字符串，例如：

```text
ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**重要：**

- 这串字符串只显示一次，一定要复制保存到安全的地方（比如密码管理器）。
- 不要提交到代码仓库、不要发到聊天群、不要贴到截图里。

### 1.4 在 Git 中使用 Token

之后在命令行使用 `git push` / `git pull` 时：

- Username：填你的 **GitHub 用户名**
- Password：填 **刚才生成的 Token**

#### 1.4.1 建议开启凭证缓存（Windows / macOS）

这样就不用每次都输入 Token。

```bash
# Mac / Linux（常见）
git config --global credential.helper store

# Windows（Git for Windows 通常支持）
git config --global credential.helper wincred
# 或
git config --global credential.helper manager-core
```

配置后，首次输入 Token 时，Git 会帮你记住，下次自动使用。

---

## 二、集群上公共代码文件夹使用规范（重要）

在集群（服务器）上通常会配置**公共代码文件夹**，供大家跑任务、调试环境等。为避免互相覆盖、产生难以追踪的问题，统一按以下规范使用：

1. 公共代码文件夹中的代码必须与远程仓库的 `origin/main` 分支保持同步，用作「只读运行环境」。
2. **严禁**在公共代码文件夹中直接修改代码（包括编辑、添加、删除文件），也**不允许在此目录下执行 `git commit`、`git push` 或创建/切换开发分支。
3. 在公共代码文件夹中，只允许执行 **`git pull origin main`** 将代码更新到 `origin/main` 的最新状态，用于保证运行环境是最新的主干代码。
4. 如需修改代码或进行开发，请在**自己的工作路径**中操作：
   - 在个人目录（例如 `~/workspace/` 或其他自定路径）执行 `git clone https://github.com/<org-or-user>/<repo>.git`；
   - 在该个人副本中按本文档「本地开发流程」进行开发、修改、测试；
   - 完成后通过 `git push` 将修改提交到远程分支；
   - 在 GitHub 上发起 Pull Request（PR），走正常的合并流程。
5. 公共代码文件夹仅作为：
   - 跑线上/集群任务的代码来源；
   - 通过 `git pull origin main` 获取已合入主干的最新代码。
6. 如发现有人误在公共代码文件夹中提交或修改代码，请及时在团队内沟通，并将目录恢复到 `origin/main` 的干净状态（需谨慎操作，避免误删未备份的数据文件）。

---

## 三、本地开发流程：克隆仓库 → 新建分支 → 提交代码

> ⚠️ 本节描述的开发流程，**只适用于个人工作路径中的仓库副本**，**不适用于集群公共代码文件夹**。公共文件夹仅允许 `git pull origin main`。

标准流程如下：

1. 克隆远程仓库
2. 拉最新主分支代码
3. 基于主分支创建功能分支
4. 在功能分支上开发、提交
5. 推送分支到远程

### 3.1 克隆仓库（在个人路径中）

在 GitHub 项目页右上角点击 `Code`，选择 **HTTPS**，复制地址，例如：

```text
https://github.com/<org-or-user>/<repo>.git
```

在本地终端执行：

```bash
# 到你想放项目的目录，比如 ~/dev
cd ~/dev

# 克隆项目
git clone https://github.com/<org-or-user>/<repo>.git

# 进入项目目录
cd <repo>
```

### 3.2 配置用户名和邮箱（只需一次）

如果第一次使用 Git，建议配置全局用户名和邮箱：

```bash
git config --global user.name "你的名字或ID"
git config --global user.email "你的邮箱（建议与 GitHub 一致）"
```

验证：

```bash
git config --global --list
```

### 3.3 拉取最新主分支（main）

进入项目目录：

```bash
cd <repo>

# 查看当前分支
git status

# 切换到 main 分支（如果是 master，则改为 master）
git checkout main

# 拉取远程 main 最新代码
git pull origin main
```

> **注意：** 有些项目主分支叫 `master`、`develop`，以实际仓库为准。

### 3.4 创建新功能分支

> 规范：每个功能 / 修复，在主分支基础上创建**独立分支**进行开发，不直接在 `main` 上修改。

命名建议：

- 功能：`feature/<简要描述>`  
  例：`feature/login-page`, `feature/user-profile-api`
- 修复：`fix/<简要描述>`  
  例：`fix/login-null-pointer`

示例：

```bash
# 确保在最新 main 上
git checkout main
git pull origin main

# 创建并切换到新分支（举例：新增登录页）
git checkout -b feature/login-page
```

### 3.5 编辑代码

在你喜欢的编辑器/IDE 中（VS Code、WebStorm、IntelliJ、Vim 等）进行修改。

例如：

- 新增文件 `src/login.js`
- 修改 `src/router.js`

…

### 3.6 查看修改状态

```bash
git status
```

会看到类似：

```text
Changes not staged for commit:
  modified:   src/router.js

Untracked files:
  src/login.js
```

### 3.7 将修改加入暂存区（git add）

将需要提交的文件添加到暂存区：

```bash
# 添加指定文件
git add src/login.js src/router.js

# 或者一次性添加当前目录所有变更（需谨慎确认）
git add .
```

再次查看：

```bash
git status
```

应显示：

```text
Changes to be committed:
  new file:   src/login.js
  modified:   src/router.js
```

### 3.8 编写提交信息并提交（git commit）

编写简洁、明确的提交信息，说明这次改动的意图。

推荐格式（示例）：

- `feat: add login page UI`
- `fix: handle null token when auto login`
- `refactor: extract user service`

提交：

```bash
git commit -m "feat: add login page UI"
```

如果希望提交信息换行、写得更详细，可以不加 `-m`，直接：

```bash
git commit
```

Git 会打开编辑器，让你写多行 commit message。

### 3.9 将本地分支推送到远程（git push）

第一次推送新分支：

```bash
git push -u origin feature/login-page
```

说明：

- `origin`：默认的远程仓库名称。
- `feature/login-page`：当前本地分支名称。
- `-u`：设置 upstream，之后可以简写为 `git push` / `git pull`。

以后在该分支上继续修改并提交后，只需：

```bash
git push
```

---

## 四、在 GitHub 上创建 Pull Request，将代码合入主分支

当你在功能分支上的开发和自测完成后，需要发起一个 **Pull Request（PR）**，让代码合入主分支（例如 `main`）。

### 4.1 在 GitHub 上创建 PR

1. 打开 GitHub 上该仓库页面。
2. 通常在你刚 `git push` 之后，页面顶部会有黄色提示条：

   > `Compare & pull request`

   直接点击即可。

   如果没有提示：

   1. 点击上方 `Pull requests` 标签。
   2. 右侧点击 `New pull request`。
   3. 在比较页面选择：
      - **base**：目标分支，通常是 `main`
      - **compare**：你的功能分支，比如 `feature/login-page`

3. 确认比较分支无误后，点击 **`Create pull request`**。

### 4.2 撰写 PR 标题与说明

**PR 标题** 建议：

- 简明扼要说明改动的主要目的。
- 建议与最终希望在 main 中保留的 commit message 类似。

例如：

- `feat: add login page and basic validation`
- `fix: handle empty user profile response`

**PR 描述** 可以包含：

- 这次改动的背景/目的
- 主要改动点（可使用列表列出）
- 对已有功能的影响
- 如何验证（测试步骤）
- 是否有已知问题/后续计划

#### 在同一 PR 上继续修改的流程：

1. 在本地当前功能分支继续改代码。
2. 提交新 commit：

   ```bash
   git add <修改的文件>
   git commit -m "fix: address review comments"
   ```

3. 推送到远程：

   ```bash
   git push
   ```

GitHub 会自动更新同一个 PR，无需重新创建。

### 4.3 合并 PR 到主分支

当以下条件满足时，一般就可以合并：

- 不存在合并冲突（conflict）。
- 代码不存在未解决的严重问题（自行检查）。

> 暂不考虑存在冲突的情况，如遇冲突可自行搜索解决方法，或团队内求助。

在 PR 页面底部通常会有一个合并按钮，具体文案可能是：

- `Squash and merge`
- `Rebase and merge`
- `Merge pull request`

> 推荐团队统一使用 **Squash and merge**，保证主干历史清晰，一 PR 一 commit。

点击合并后，GitHub 会：

- 将你分支上的改动合入目标分支（如 `main`）。
- 关闭该 PR。

### 4.4 删除源分支（推荐）

合并成功后，GitHub 通常会在 PR 页面提示：

> `Delete branch`

建议点击删除：

- 删除的是远程的功能分支（例如 `origin/feature/login-page`）。
- 已经合入 `main` 的代码不会受影响。
- PR 记录（讨论、diff）仍然保留。

本地如需删除对应分支：

```bash
# 切回 main
git checkout main
git pull origin main

# 删除本地已合并的功能分支
git branch -d feature/login-page
```

---

## 五、完整协作流程示例（命令速查）

> 再次强调：下面的流程示例适用于 **个人路径** 的仓库，不适用于集群公共代码文件夹。

```bash
# 1. 克隆仓库（第一次，在个人目录）
git clone https://github.com/<org-or-user>/<repo>.git
cd <repo>

# 2. 更新主分支
git checkout main
git pull origin main

# 3. 创建并切到新分支（以新增登录页面为例）
git checkout -b feature/login-page

# 4. 开发：编辑代码...

# 5. 查看变更
git status

# 6. 添加到暂存区
git add .

# 7. 提交
git commit -m "feat: add login page UI"

# 8. 推送到远程并关联
git push -u origin feature/login-page
```

然后在 GitHub：

1. 创建 PR（base: `main`, compare: `feature/login-page`）。
2. 填写标题和说明。
3. 检查 PR 的代码修改无误且无冲突。
4. 合并 PR（推荐 Squash and merge）。
5. 删除远程功能分支。

本地清理：

```bash
git checkout main
git pull origin main
git branch -d feature/login-page
```

---

## 六、附录：常见问题（FAQ）

### Q1：提交时报错 “Please tell me who you are”

说明未配置 Git 用户名/邮箱，按前文 3.2 配置：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Q2：`git push` 时提示认证失败（Authentication failed）

可能原因：

- 使用了 GitHub 密码而不是 Token。
- Token 权限不足或已过期。

解决：

1. 在 GitHub 重新生成 Personal Access Token (classic)。
2. 清理旧凭证并重新输入：

   ```bash
   # Mac / Linux
   git config --global --unset credential.helper

   # 或在系统的凭证管理器里删除旧记录，再重新 git push
   ```

3. 按提示重新输入用户名和新 Token。

### Q3：不小心在 main 分支上开发了

如果已经写了一些代码还没提交：

```bash
# 基于当前 main 创建分支并保留修改
git checkout -b feature/xxx
# 然后在新分支上继续正常 add/commit/push
```

如果已经在 main 上 commit 了，且没有 push：

```bash
# 在当前 main 上创建新分支并保留历史
git checkout -b feature/xxx

# 回到 main 并回滚到远程状态（注意：会丢弃本地 commit，谨慎）
git checkout main
git reset --hard origin/main
```

之后在 `feature/xxx` 分支上继续开发并提 PR 即可。

### Q4：不小心在集群公共代码文件夹中修改/提交了怎么办？

1. 先确认是否有需要保留的代码改动，如果有：
   - 可以将修改过的文件复制到你个人工作目录下的仓库；
2. 在公共代码文件夹中恢复到干净状态（**注意：会丢弃未保存的修改**）：

   ```bash
   git checkout main
   git fetch origin
   git reset --hard origin/main
   git clean -fd
   ```

3. 在个人路径中重新按开发流程进行修改、提交、发 PR。  
4. 如有不确定的情况，建议先在团队内沟通后再操作。
