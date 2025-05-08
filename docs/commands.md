## ✅ 多次执行命令
``for i in {1..10}; do echo "Run #$i";go test -run 3A; done``

这个命令是在 Bash（或类 Bash 的 shell，比如 zsh）中执行一个循环，用来重复运行 `go test -run 3A` 十次，并打印每次运行的编号。

---

### **逐部分解释：**

#### 1. `for i in {1..10}`

* 这是一个 **Bash 循环结构**。
* `i` 是循环变量，会依次取值：`1, 2, ..., 10`。
* 花括号 `{1..10}` 是 Bash 的 **序列生成语法**，表示从 1 到 10。

#### 2. `do ... done`

* `do` 开始定义循环体（要执行的语句块）。
* `done` 表示循环体结束。

#### 3. `echo "Run #$i"`

* `echo` 用于输出字符串。
* `"Run #$i"` 会输出类似 `Run #1`、`Run #2`，其中 `$i` 是当前循环次数的变量值。

#### 4. `go test -run 3A`

* 这是你要执行的 Go 测试命令。
* `-run 3A` 让 `go test` 只运行名字中 **包含正则匹配 `3A`** 的测试函数，比如 `TestFoo3A`、`Test3Alpha` 等。
* `go test` 会根据正则匹配函数名，并只执行匹配的测试。

---

### **执行流程示例：**

第一轮时：

```bash
i = 1
echo "Run #1"
go test -run 3A
```

然后进入第二轮，直到第十轮。

---

## ✅ `git switch -c new-branch-name`

### 🔍 逐个解释：

* `git`
  Git 的命令行工具调用入口。

* `switch`
  Git 2.23 之后新增的子命令，用于**切换分支**，更语义化的替代 `checkout`。

* `-c`
  是 `--create` 的缩写，表示**创建并切换到**一个新的分支。

* `new-branch-name`
  是你想创建的新分支的名称。例如：`feature/login`、`bugfix/123` 等。

📌 这个命令的意思是：**创建一个名为 `new-branch-name` 的新分支，并立即切换到它**。

---

## ✅ 命令：`git checkout -b new-branch-name`

### 🔍 逐个解释：

* `git`
  Git 工具命令前缀。

* `checkout`
  旧的多用途命令，用于：

  * 切换分支（最常见用法之一）
  * 恢复工作区的文件
  * 创建并切换分支（和 `-b` 一起）

* `-b`
  是 `--branch` 的缩写，表示**创建一个新分支**。

* `new-branch-name`
  要创建的分支的名称。

📌 这个命令的意思是：**创建并切换到新分支**（功能和 `git switch -c` 相同，但语法更早期）。

---

## ✅ 命令：`git switch existing-branch-name`

### 🔍 逐个解释：

* `git`
  Git 命令调用。

* `switch`
  表示切换分支。

* `existing-branch-name`
  是一个**已经存在的本地分支名称**。

📌 表示：**直接切换到已有分支**，不会创建新分支。

---

## ✅ 命令：`git checkout existing-branch-name`

与上面完全相同，区别是：

* 用的是 `checkout` 命令（老语法）
* 在 Git 2.23 之前用得最广泛，现在推荐使用 `switch` 替代

---

## ✅ 命令：`git branch`

### 🔍 解释：

* `branch` 是用于查看、创建、删除本地分支的子命令。
* **单独使用**时表示：**列出当前所有本地分支**，当前所在分支前会有 `*` 标记。

---

## ✅ 命令：`git branch -r`

### 🔍 解释：

* `-r` 是 `--remotes` 的缩写，表示**查看远程分支列表**。
* 用于知道远程仓库（如 `origin`）有哪些分支可以拉取。

---

在 Git 中，“合并”是指将一个分支的修改合并到当前分支，**常用于功能开发完成后把子分支合并回主分支（如 `main` 或 `dev`）**。

以下是 Git 合并的常用命令和详细解释：

---

## ✅ 基本合并命令

```bash
git merge other-branch
```

### 🔍 逐个解释：

* `git`：Git 命令调用入口。
* `merge`：子命令，表示合并。
* `other-branch`：你想合并进当前分支的目标分支名。

📌 含义：**把 `other-branch` 的修改合并到当前分支上。**

### ⚠️ 注意：

你需要**先切换到接收合并的分支**上（通常是主分支 `main` 或 `dev`）：

```bash
git switch main         # 切换到目标分支
git merge feature/foo   # 将 feature/foo 分支合并进 main
```
---

## 📌 处理冲突

如果你在 `git merge` 时遇到冲突（Git 会提示 `CONFLICT`）：

1. 手动打开有冲突的文件，编辑冲突标记区域：

   ```diff
   <<<<<<< HEAD
   当前分支的内容
   =======
   被合并分支的内容
   >>>>>>> other-branch
   ```

2. 保存修改后，执行：

   ```bash
   git add conflicted-file
   git commit   # 完成合并，Git 会生成一个合并提交
   ```

---

## 🧼 常用辅助命令

### 1. 查看分支图（可视化合并）

```bash
git log --oneline --graph --all
```

### 2. 取消合并（未提交时）

```bash
git merge --abort
```

---

## 🔄 示例流程

```bash
git switch main           # 切换到 main 分支
git merge feature/login   # 将 feature/login 合并到 main
```

---

Git 的“回退”命令用于撤销更改、恢复文件或重置提交。根据你想回退的**内容和场景不同**，使用的命令也不同。以下是几种常见的回退操作及其解释：

---

## ✅ 1. 回退 **工作区文件改动**（还没 add）

```bash
git checkout -- filename
```

* 恢复 `filename` 文件到上次提交的状态。
* ⚠️ 改动会丢失，**不能恢复**。

或使用新版命令：

```bash
git restore filename
```

---

## ✅ 2. 回退 **暂存区（已 add）改动**

```bash
git reset HEAD filename
```

* 把 `filename` 从暂存区移回工作区，**保留你文件中的改动**。
* 相当于取消 `git add`。

---

## ✅ 3. 回退 **最近一次提交，但保留改动（软回退）**

```bash
git reset --soft HEAD^
```

* 回退上一次提交，但保留更改和暂存状态（add 的内容还在）。
* 用于 “撤销提交但想重写提交信息” 的场景。

---

## ✅ 4. 回退提交并保留工作区文件（混合回退，默认）

```bash
git reset HEAD^
```

或显式写法：

```bash
git reset --mixed HEAD^
```

* 回退提交，同时取消暂存，改动保留在工作区。
* 是最常用的 reset 方式。

---

## ✅ 5. 回退提交并删除工作区改动（硬回退❗）

```bash
git reset --hard HEAD^
```

* 回退一次提交，且连工作区文件都还原。
* ⚠️ **危险操作，改动会永久丢失**，不可恢复。

---

## ✅ 6. 回退到指定 commit

```bash
git reset --hard <commit-id>
```

* 把分支指针和文件状态都重置到 `<commit-id>`。
* 用 `git log --oneline` 可以查看 commit-id。

---

## ✅ 7. 创建撤销提交（保留历史）→ `git revert`

```bash
git revert <commit-id>
```

* 用于“撤销某次提交”，但以一次新的提交方式记录下来。
* 常用于**公共分支**，因为不会更改历史。

---

## 总结对比

| 操作目的                 | 命令                      | 是否可恢复 |
| ------------------------ | ------------------------- | ---------- |
| 恢复未暂存的文件改动     | `git restore filename`    | 否         |
| 取消已暂存的文件         | `git reset HEAD filename` | 是         |
| 撤销最近提交（保留改动） | `git reset --soft HEAD^`  | 是         |
| 撤销提交并清暂存         | `git reset --mixed HEAD^` | 是         |
| 撤销提交+改动都清除❗     | `git reset --hard HEAD^`  | 否⚠️        |
| 撤销指定提交（不改历史） | `git revert <commit-id>`  | 是         |

---
`git rebase` 是 Git 中一个非常强大的命令，它的作用是：**改变提交历史的“基础”位置**，从而让分支历史更“线性”、更干净。

---

## ✅ 基本语法

```bash
git rebase <base-branch>
```

📌 表示：**把当前分支的提交“移到” `<base-branch>` 的最新提交之后**，实现一种“像是从该分支分出来一样”的效果。

---

## 🎯 使用场景举例

### 场景：你在 `feature` 分支开发，`main` 分支已经更新了。

你想让 `feature` 基于 `main` 最新的内容继续开发，这时可以：

```bash
git switch feature
git rebase main
```

作用是：把 `feature` 上的提交“剪下来”，再“贴”到 `main` 最新的提交之后。

---

## 🔁 和 `merge` 的区别

| 比较项         | `git merge`                         | `git rebase`                     |
| -------------- | ----------------------------------- | -------------------------------- |
| 提交历史       | 非线性，会保留分支记录（分叉+合并） | 线性，看起来像一条线             |
| 是否产生新提交 | 是（合并提交）                      | 否（除非有冲突或 `-i` 重写历史） |
| 是否更改历史   | 否                                  | 是（会更改 commit ID）           |
| 推荐场景       | 公共分支、多人协作                  | 私人分支、整理提交历史           |

---

## ✍️ 示例图解（原始状态）：

```
main:    A---B---C
                   \
feature:            D---E
```

执行：

```bash
git switch feature
git rebase main
```

变成：

```
main:    A---B---C
                        \
feature:                 D'---E'   ← D/E 被“复制”到 C 后
```

---

## ⚠️ 注意事项

* `rebase` 会**重写提交历史**，因此**不要在已经 push 到共享仓库的分支上 rebase**，否则会导致别人 pull 失败。
* 如果有冲突，Git 会暂停并提示你解决冲突，解决后执行：

```bash
git add conflicted-file
git rebase --continue
```

也可以中断整个 rebase：

```bash
git rebase --abort
```

---

## 🔧 进阶：交互式 rebase（整理提交）

```bash
git rebase -i HEAD~3
```

* `-i` 表示 **interactive**（交互式）
* `HEAD~3` 表示最近 3 次提交

你可以在弹出的编辑器中选择：

* `pick`：保留提交
* `reword`：修改提交说明
* `squash`：合并多个提交
* `drop`：丢弃提交

---

## ✅ 小结

| 用途                 | 命令示例                |
| -------------------- | ----------------------- |
| 当前分支基于主分支   | `git rebase main`       |
| 清理最近几次提交     | `git rebase -i HEAD~3`  |
| 遇到冲突后继续       | `git rebase --continue` |
| 中断回滚 rebase 操作 | `git rebase --abort`    |

---





