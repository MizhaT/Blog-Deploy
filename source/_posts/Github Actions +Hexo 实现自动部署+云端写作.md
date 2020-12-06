---
title: Github Actions +Hexo 实现自动部署+云端写作
categories: Hexo
tags: Hexo
date: 2020-12-5 12:00:00
cover: 
---
# 参考
[GitHub Action + Hexo实现在线写作](https://fushaolei.github.io/2020/09/21/github-action-hexo-online/)

## 前言
Hexo每次修改、写一篇博文的时候，总是要经过
```
hexo clean
hexo g
hexo d
```
实在是太麻烦了啊啊啊
那么，写一个bat？
```
@echo off
call hexo clean
call hexo g -d
pause
```
自己电脑不在身边的情况下呐？ = =
那么用Github Actions +Hexo 实现自动部署+云端写作

## 使用github aciton实现自动部署

### 公钥
桌面右键 打开 Git Bash Here

```
ssh-keygen -t rsa -C "注册的邮箱"

cd C:\Users\用户名\.ssh #（如果没有执行第2步，则不会有这个文件夹）

cat id_rsa.pub  #公钥

cat id_rsa  #私钥，稍后要用上
```

![.ssh](https://cdn.jsdelivr.net/gh/MizhaT/Pic@main/2020/12/6/Github_Actions_+Hexo_实现自动部署+云端写作/ssh.png)

复制，打开github ，点自己头像 >> settings >> SSH and GPG keys >>New SSH key 粘贴公钥

### 新建仓库
将
```
_config.yml
package.json
package-lock.json
themes
sourc
scaffolds
```
上传到仓库（不能留下.git文件）

### 私钥
仓库 --> setting --> secrets --> New repository secret
填入 私钥
![secrets](https://cdn.jsdelivr.net/gh/MizhaT/Pic@main/2020/12/6/Github_Actions_+Hexo_实现自动部署+云端写作/secrets.png)

### Actions
仓库 --> Actions --> set up a workflow yourself
替换为
```
# workflow name
name: Hexo Blog CI

# master branch on push, auto run
on: 
  push:
    branches:
      - main
      
jobs:
  build: 
    runs-on: ubuntu-latest 
        
    steps:
    # check it to your workflow can access it
    # from: https://github.com/actions/checkout
    - name: Checkout Repository master branch
      uses: actions/checkout@master 
      
    # from: https://github.com/actions/setup-node  
    - name: Setup Node.js 12.x 
      uses: actions/setup-node@master
      with:
        node-version: "12.x"
    
    - name: Setup Hexo Dependencies
      run: |
        npm install hexo-cli -g
        npm install
    
    - name: Setup Deploy Private Key
      env:
        HEXO_DEPLOY_PRIVATE_KEY: ${{ secrets.HEXO_DEPLOY_PRIVATE_KEY }}
      run: |
        mkdir -p ~/.ssh/
        echo "$HEXO_DEPLOY_PRIVATE_KEY" > ~/.ssh/id_rsa 
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan github.com >> ~/.ssh/known_hosts
        
    - name: Setup Git Infomation
      run: | 
        git config --global user.name '<你的用户名>' 
        git config --global user.email '<你登录的email>'
    - name: Deploy Hexo 
      run: |
        hexo clean
        hexo generate 
        hexo deploy
```![enter description here](https://cdn.jsdelivr.net/gh/MizhaT/Pic@main/2020/12/6/Github_Actions_+Hexo_实现自动部署+云端写作/secrets.png)
大功告成
