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
git rm --cached file1
```