# GitHub Pages 部署指南

## 步骤 1：创建 GitHub 仓库

1. 访问 https://github.com/new
2. 创建一个名为 `docs-public` 的仓库
3. **重要**：仓库设置为 **Public**（公开）
4. 不要初始化 README

## 步骤 2：推送代码到 GitHub

```bash
cd docs-public
git remote add origin https://github.com/YOUR_USERNAME/docs-public.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

**记得把 `YOUR_USERNAME` 替换成你的 GitHub 用户名！**

## 步骤 3：启用 GitHub Pages

1. 进入仓库的 **Settings**（设置）
2. 左侧菜单找到 **Pages**
3. **Build and deployment** → **Source** 选择 **GitHub Actions**
4. 保存

## 步骤 4：配置 baseURL

编辑 `hugo.toml` 文件，将第 1 行的 `YOUR_USERNAME` 替换成你的 GitHub 用户名：

```toml
baseURL = 'https://YOUR_USERNAME.github.io/docs-public/'
```

## 步骤 5：推送更新

```bash
git add .
git commit -m "Update baseURL"
git push
```

## 完成！

GitHub Actions 会自动构建并部署你的网站。几分钟后访问：

```
https://YOUR_USERNAME.github.io/docs-public/
```

## 查看部署状态

1. 仓库页面点击 **Actions** ���签
2. 查看最新的 workflow 运行状态
3. 等待变为绿色的 ✅

## 更新网站

以后每次推送代码到 `main` 分支，GitHub Actions 会自动重新构建并部署：

```bash
git add .
git commit -m "Update content"
git push
```
