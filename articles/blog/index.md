---
title: "5分钟无服务器部署个人博客"
date: "2023-01-03"

---

### 说明
- 使用Github Pages托管个人博客，不需要服务器，不需要克隆到本地。内容用markdown，方便迁移。
- 代码作者[mebtte](https://github.com/mebtte/animal-photosynthesis)


### 快速部署
- 拷贝项目。可直接 fork [我的项目](https://github.com/tomatocuke/tomatocuke.github.io)到你自己的仓库，更改为你自己的`<username>.github.io`，否则无法使用。
- 随便更改一下`article`里的文件并提交，以触发`.github/workflows/gh_pages.yml`，在`Actions`中查看进度。(须等待约1分钟)
- 进入 `Settings` > `Pages`，更改`Branch`为`gh-pages`分支并保存. 此时可以看到你的网站地址 `https://<username>.github.io` (稍等1分钟后可以访问)

### 自定义域名(可选)
- github的用户名及其难起，托管域名也就不好记，所以改为自定义域名。
- 在云平台解析你所拥有的域名，记录类型为CNAME，记录值为你的托管域名`<username>.github.io`。
- 进入 `Settings` > `Pages`，更改`Custom domain` 为你所解析的域名，系统自动检查DNS解析。
- 等待片刻后可以访问。若不成功有其他问题请参考[官方文档](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)

### 配置与内容
- 修改配置。`script/config.js`
- 写博客。以单独目录形式放在`article`目录下。
- 下方Discussions 在 Settings 里开通，可以直接点击博客进行编辑。 小蓝鸟如果不想要在`src/template/article/article_action.ejs`里注释掉。
- 其实不需要克隆到本地，若本地使用`npm install && npm run build` 后生成html文件在build目录下
