- # Git Hooks 与 Pre-commit 配置指南

  ## 📋 概述

  本指南介绍两种为Git仓库添加提交前检查的方法：
  1. **原生Git Hooks**：手动编写Shell脚本
  2. **Pre-commit框架**：使用YAML配置管理多种代码检查工具

  ## 🚀 快速开始

  ### 方法1：原生Git Hooks（简单直接）

  #### 1.1 创建pre-commit钩子
  ```bash
  # 进入项目的.git/hooks目录
  cd /e/GITHUB/NineSevenFiveOne/.git/hooks
  
  # 创建pre-commit文件（Windows可用文本编辑器创建）
  touch pre-commit
  
  # 添加执行权限
  chmod +x pre-commit
  ```

  #### 1.2 编辑pre-commit内容
  使用文本编辑器打开`pre-commit`文件，添加以下内容：

  ```bash
  #!/bin/bash
  
  echo "🔍 开始Git提交前检查..."
  echo "================================"
  
  # 获取当前分支名
  current_branch=$(git symbolic-ref --short HEAD)
  
  # 检查是否提交到受保护分支
  protected_branches="^(main|master|develop)$"
  if [[ $current_branch =~ $protected_branches ]]; then
      echo "❌ 错误：不允许直接提交到受保护分支 '$current_branch'"
      echo "💡 请创建特性分支：git checkout -b feature/your-feature"
      exit 1
  fi
  
  # 检查是否有调试语句
  if git diff --cached --name-only | xargs grep -n "console\.log\|print(" 2>/dev/null; then
      echo "⚠️  警告：提交中包含调试语句"
      echo "是否继续提交？(y/N)"
      read -r response
      if [[ ! "$response" =~ ^([yY][eE][sS]|[yY])$ ]]; then
          exit 1
      fi
  fi
  
  # 检查文件大小（大于1MB的文件）
  large_files=$(git diff --cached --name-only --diff-filter=ACM | xargs ls -l 2>/dev/null | awk '$5 > 1048576 {print $NF}')
  if [ ! -z "$large_files" ]; then
      echo "⚠️  警告：以下文件超过1MB："
      echo "$large_files"
      echo "是否继续提交？(y/N)"
      read -r response
      if [[ ! "$response" =~ ^([yY][eE][sS]|[yY])$ ]]; then
          exit 1
      fi
  fi
  
  echo "✅ 检查完成，可以提交！"
  echo "================================"
  exit 0
  ```

  #### 1.3 测试钩子
  ```bash
  # 回到项目根目录
  cd /e/GITHUB/NineSevenFiveOne
  
  # 创建测试文件
  echo "console.log('测试')" > test.js
  
  # 尝试提交
  git add test.js
  git commit -m "测试原生钩子"
  # 应该会看到钩子的检查输出
  ```

  ### 方法2：Pre-commit框架（推荐）

  #### 2.1 安装pre-commit
  ```bash
  # 全局安装（最简单）
  pip install pre-commit
  
  # 或使用pipx（更隔离）
  pipx install pre-commit
  ```

  #### 2.2 创建配置文件
  在项目根目录创建`.pre-commit-config.yaml`：

  ```yaml
  # .pre-commit-config.yaml
  repos:
    # 基础代码质量检查
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v4.4.0
      hooks:
        - id: trailing-whitespace      # 删除行尾空格
        - id: end-of-file-fixer        # 确保文件以换行符结束
        - id: check-yaml               # 检查YAML语法
        - id: check-json               # 检查JSON语法
          args: ['--allow-empty']      # 允许空JSON文件
        - id: check-added-large-files  # 禁止大文件（默认>5MB）
          args: ['--maxkb=1024']       # 自定义为1MB
        - id: check-merge-conflict     # 检查合并冲突标记
        - id: check-symlinks           # 检查损坏的符号链接
        - id: detect-private-key       # 检查私钥文件
  
    # 通用文本文件检查
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v4.4.0
      hooks:
        - id: forbid-new-submodules    # 禁止新增子模块
        - id: no-commit-to-branch      # 禁止提交到特定分支
          args: ['--branch', 'main', '--branch', 'master']
  
    # Markdown文件检查
    - repo: https://github.com/igorshubovych/markdownlint-cli
      rev: v0.35.0
      hooks:
        - id: markdownlint
          args: ['--fix']  # 自动修复问题
  
    # 如果项目有Python文件，可以启用以下检查
    # - repo: https://github.com/astral-sh/ruff-pre-commit
    #   rev: v0.1.0
    #   hooks:
    #     - id: ruff
    #       args: [--fix, --exit-non-zero-on-fix]
    #     - id: ruff-format
  
    # 如果项目有前端文件，可以启用以下检查
    # - repo: https://github.com/pre-commit/mirrors-prettier
    #   rev: v3.0.0
    #   hooks:
    #     - id: prettier
    #       types_or: [javascript, typescript, css, scss, html, json, markdown]
  ```

  #### 2.3 安装并激活钩子
  ```bash
  # 在项目根目录执行
  cd /e/GITHUB/NineSevenFiveOne
  
  # 安装钩子到.git目录
  pre-commit install
  
  # 可选：安装为提交时的钩子
  pre-commit install --hook-type commit-msg
  
  # 验证安装
  pre-commit --version
  
  # 查看已安装的钩子
  pre-commit list
  ```

  #### 2.4 测试pre-commit
  ```bash
  # 方法1：对暂存区的文件运行检查
  git add .
  pre-commit run
  
  # 方法2：对所有文件运行检查
  pre-commit run --all-files
  
  # 方法3：测试特定钩子
  pre-commit run trailing-whitespace --all-files
  
  # 实际提交测试
  echo "# 测试文件" > README.md
  git add README.md
  git commit -m "测试pre-commit配置"
  # 提交时会自动运行检查
  ```

  ## 🔧 高级配置

  ### 3.1 跳过钩子检查
  ```bash
  # 临时跳过所有钩子
  git commit --no-verify -m "紧急提交"
  
  # 跳过特定钩子类型
  SKIP=trailing-whitespace git commit -m "跳过空格检查"
  ```

  ### 3.2 更新工具版本
  ```bash
  # 更新所有钩子到最新版本
  pre-commit autoupdate
  
  # 更新特定仓库的版本
  pre-commit autoupdate --repo https://github.com/pre-commit/pre-commit-hooks
  ```

  ### 3.3 自定义本地钩子
  ```bash
  # 在.pre-commit-config.yaml中添加本地钩子
  - repo: local
    hooks:
      - id: custom-check
        name: "自定义检查"
        entry: ./scripts/custom-check.sh
        language: script
        files: \.py$
  ```

  ### 3.4 配置文件示例
  创建更完整的配置示例文件：

  ```yaml
  # .pre-commit-config.example.yaml
  # 这是一个完整的pre-commit配置示例
  # 复制为 .pre-commit-config.yaml 并根据需要修改
  
  repos:
    # 基本代码质量
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v4.4.0
      hooks:
        - id: trailing-whitespace
        - id: end-of-file-fixer
        - id: check-yaml
        - id: check-json
  
    # 安全性检查
    - repo: https://github.com/gitleaks/gitleaks
      rev: v8.18.0
      hooks:
        - id: gitleaks
          args: ['--verbose', '--redact']
  
    # Markdown检查
    - repo: https://github.com/igorshubovych/markdownlint-cli
      rev: v0.35.0
      hooks:
        - id: markdownlint
          args: ['--fix', '--ignore', 'node_modules']
  
  ci:
    skip: [gitleaks]  # CI环境中跳过某些检查
  ```

  ## 📁 项目结构建议

  ```
  NineSevenFiveOne/
  ├── .pre-commit-config.yaml      # pre-commit配置
  ├── .git/hooks/pre-commit        # 原生钩子（由pre-commit生成）
  ├── scripts/
  │   └── custom-check.sh          # 自定义检查脚本
  ├── README.md
  └── (其他项目文件)
  ```

  ## 🚨 故障排除

  ### 常见问题

  1. **钩子不生效**
     ```bash
     # 检查文件权限
     ls -la .git/hooks/pre-commit
     
     # 重新安装
     pre-commit uninstall
     pre-commit install
     ```

  2. **pre-commit命令未找到**
     ```bash
     # 检查安装
     pip show pre-commit
     
     # 确保在PATH中
     which pre-commit
     ```

  3. **检查太慢**
     ```bash
     # 只检查暂存文件（默认）
     git add 修改的文件
     git commit  # 只检查修改的文件
     
     # 或排除大文件
     # 在.pre-commit-config.yaml中添加exclude
     ```

  4. **需要调试**
     ```bash
     # 详细输出
     pre-commit run --verbose
     
     # 调试模式
     PRE_COMMIT_COLOR=never pre-commit run
     ```

  ## 📝 提交到你的仓库

  ### 4.1 添加配置文件
  ```bash
  cd /e/GITHUB/NineSevenFiveOne
  
  # 添加配置文件到版本控制
  git add .pre-commit-config.yaml
  
  # 可选：添加示例配置
  git add .pre-commit-config.example.yaml
  
  # 提交
  git commit -m "添加git hooks和pre-commit配置"
  
  # 推送到远程
  git push 9751 main
  ```

  ### 4.2 添加文档说明
  创建`DEVELOPMENT.md`或更新`README.md`：

  ```markdown
  ## 开发设置
  
  ### 安装pre-commit钩子
  
  本项目使用pre-commit进行代码质量检查。设置步骤如下：
  
  1. 安装pre-commit：
     ```bash
     pip install pre-commit
  ```

  2. 安装Git钩子：
     ```bash
     pre-commit install
     ```

  3. （可选）对所有文件运行一次检查：
     ```bash
     pre-commit run --all-files
     ```

  ### 跳过检查

  紧急情况下可跳过检查：
  ```bash
  git commit --no-verify -m "紧急提交"
  ```

  ### 更新钩子

  ```bash
  pre-commit autoupdate
  ```
  ```
  
  ## 🎯 最佳实践建议
  
  1. **团队协作**：将`.pre-commit-config.yaml`加入版本控制
  2. **渐进采用**：先启用基础检查，逐步添加复杂规则
  3. **CI集成**：在CI流水线中也运行`pre-commit run --all-files`
  4. **定期更新**：每月运行`pre-commit autoupdate`更新工具
  
  ---
  
  现在你可以测试这个配置了！运行`git add`和`git commit`时会自动触发检查。如果有任何问题，查看`.git/hooks/pre-commit`文件的内容或运行`pre-commit run --verbose`查看详细输出。