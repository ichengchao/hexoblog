---
title: 云原生Blog搭建
date: 2020-06-12 11:27:12
tags:

 - tech
 - blog
 - github
 - aliyun
 - hexo
---



<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_06_12_15_56_1591948595175.png" alt="image-20200612155634707" style="zoom:50%;" />

### 背景

之前简单介绍过我[个人网站的架构](http://blog.chengchao.name/2018/07/03/website-architecture/),这次再次升级Blog的架构,蹭个热度,变成云原生Blog.其实之前[完全托管在Github](http://ichengchao.github.io/archive.html)的方式还是挺不错的,只是访问速度时好时坏.随着Github发布了Actions,能玩的可能性就很多了.下面介绍一下我的玩法.我用到的组件

- Hexo: 静态Blog生成工具
- Next: 比较有名的Hexo主题
- Github: 源码托管+Actions
- OSS: web服务



### Here we go

#### 1.Hexo

在本地安装一个hexo环境,并初始化好一个最简单的blog,简单几部就能完成

```sh
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

访问 http://localhost:4000 试试,一个最简单的blog就在本地生成好了,有一点建议,就是.gitignore的配置

```
.DS_Store
.vscode/
node_modules/
public/
db.json
package-lock.json
```

还有我用的是[Next](https://theme-next.iissnan.com/)的主题,算是hexo中比较有名的.

#### 2.Github

创建一个Github项目,把刚才生成好的Blog上传上去.接着设置Github Action,这个就不具体介绍了,很好用的一个东西,可以理解成是Github官方的CI/CD工具. 直接贴一下我的action的yml文件

```yaml
# deploy hexo blog to aliyun oss
name: Deploy Blog
on:
  push:
    branches: [master]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      # 配置node环境
      - uses: actions/setup-node@v1
        with:
          node-version: "12"
          
      # 配置hexo环境,并build成html
      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i
          hexo
          hexo generate
          
      # 配置oss环境
      - name: config ossutil
        uses: manyuanrong/setup-ossutil@v1.0
        with:
          endpoint: ${{ secrets.ALIYUN_OSS_ENDPOINT }}
          access-key-id: ${{ secrets.ALIYUN_OSS_KEY_ID }}
          access-key-secret: ${{ secrets.ALIYUN_OSS_KEY_SECRET }}
          
      # 把生成好的html上传到oss
      - name: upload to oss
        run: |
          ossutil rm oss://chengchaoblog/ -a -r -f
          ossutil cp public oss://chengchaoblog -r -f
```

这样当你提交了新的文章的时候就能自动build成html,并且把html上传到oss

#### 3.OSS

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_06_12_16_31_1591950660512.png" style="zoom:50%;border-style: dashed;border-width: thin;" />

这样一个速度快,而且高可用的Blog就搭建完了.我同时还设置了自定义域名+证书托管.这样一个自己域名的的https的blog就搞定了.成本几乎为零. 🌼