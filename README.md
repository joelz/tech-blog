# tech blog

## TODO
1. 修改 hugo.yaml 中的元数据
2. 清理 posts
3. typora 到 hugo 转换脚本

## Hugo 相关

安装 Hugo：`brew install hugo`

初始化新站点：`hugo new site tech-blog`

### 第一次安装themes/diary

```bash
# If your website is using Git as version control, please do as follows:
# Fetch the theme dir:
# From the root of your Hugo site, open the terminal and execute:
git submodule add https://github.com/AmazingRise/hugo-theme-diary.git themes/diary

# Then update the git repository from the root of your site:
git submodule update --remote --merge

# Run example site.
cd themes/diary/exampleSite:
hugo server --themesDir ../..

# commit .gitmodules and another file into root repo!
```

### 更新themes/diary

```bash
# Then update the git repository from the root of your site:
git submodule update --remote --merge

# Run example site.
cd themes/diary/exampleSite
hugo server --themesDir ../..

# commit gitmodule change into root repo!
```

### 将静态文件发布到 XXX.github.io/tech-blog/ 

1. hugo.yaml 中设置 `baseURL: https://XXX.github.io/tech-blog/`
2. md 文件中引用图片全部需要改为类似 `"/tech-blog/images/chinese.jpg"`
3. tech-blog 这个 repo 必须要是 public，才能**免费**发布到 github pages
4. 在 repo 的 Settings 中 GitHub Pages 标签中，指定分支和目录，开启该项目的 GitHub Pages。[多项目部署在同一个GitHub Pages](https://www.cnblogs.com/dev2007/p/13947333.html)