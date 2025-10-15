# NineSevenFiveOne
## 📋 常用 Git 命令
### 基础操作
```
# 克隆仓库
git clone <仓库地址>

# 查看状态
git status

# 添加文件
git add <文件名>
git add .              # 添加所有文件

# 提交更改
git commit -m "提交说明"

# 查看提交历史
git log
```
### 分支管理
```
# 查看分支
git branch
git branch -a          # 查看所有分支

# 创建切换分支
git branch <分支名>
git checkout <分支名>
git switch <分支名>

# 合并分支
git merge <分支名>

# 删除分支
git branch -d <分支名>
```
### 远程协作
```
# 推送代码
git push origin <分支名>

# 拉取更新
git pull origin <分支名>

# 查看远程仓库
git remote -v
```
### 撤销操作
```
# 撤销工作区修改
git restore <文件名>

# 撤销暂存区文件
git restore --staged <文件名>

# 重置提交
git reset --hard <提交ID>
```
