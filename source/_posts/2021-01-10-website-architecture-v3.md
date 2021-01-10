---
title: 个人网站架构V3
date: 2021-01-10 16:03:48
tags:

 - tech
 - website

---

<img src="/Users/chengchao/Library/Application Support/typora-user-images/image-20210110160439663.png" alt="image-20210110160439663" style="zoom:50%;" />

> 在[之前的基础](https://blog.chengchao.name/2018/07/03/website-architecture/)上做了些改进

## Update

### static page

oss 开启版本控制(之前开启静态页面的bucket不支持版本控制)

<img src="/Users/chengchao/Library/Application Support/typora-user-images/image-20210110163547121.png" alt="image-20210110163547121" style="zoom:50%;" />

### blog

这个模块的变化比较大,从原来的docker迁移到aliyun oss,编译&部署依赖github actions完成,整个过程非常自动化.只要commit就会自动部署,进一步降低本地系统的依赖

```yaml
# deploy hexo blog to aliyun oss

name: Deploy Blog

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - uses: actions/setup-node@v1
        with:
          node-version: "12"

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i
          hexo
      - name: Run a multi-line script
        run: |
          hexo generate
          ls -l public
      - name: config ossutil
        uses: manyuanrong/setup-ossutil@v2.0
        with:
          endpoint: ${{ secrets.ALIYUN_OSS_ENDPOINT }}
          access-key-id: ${{ secrets.ALIYUN_OSS_KEY_ID }}
          access-key-secret: ${{ secrets.ALIYUN_OSS_KEY_SECRET }}

      - name: upload to oss
        run: |
          ossutil rm oss://chengchaoblog/ -a -r -f
          ossutil cp public oss://chengchaoblog -r -f
```

