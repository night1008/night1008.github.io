### 回滚提交

```sh
git reset HEAD~

git push origin branchName -f
```
---

### 使用 git-token 下载仓库
```
git clone https://<user-name>:<git-token>@<github-path.git>
```

---

### 对比文件
```
git diff --no-index file1 file2
```

---

### 让 git 停止跟踪（但保留本地文件）
```
git rm -r --cached file1
```

---

### .gitmodules 子模块
> 子模块锁定的是一个具体的 commit hash，不会随上游变动

```sh
# 添加子模块
git submodule add https://github.com/xxx/yyy.git libs/yyy

# 克隆含子模块的仓库
git clone --recurse-submodules <repo-url>

# 已克隆但还没初始化子模块
git submodule update --init --recursive

# 更新所有子模块到最新
git submodule update --remote
```

```
[submodule "libs/awesome-lib"]
    path = libs/awesome-lib
    url = https://github.com/someone/awesome-lib.git
    branch = main
```