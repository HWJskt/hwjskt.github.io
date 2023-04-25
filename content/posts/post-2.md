+++
title="blog Deployment using Github Pages"
date=2023-04-25
template ="posts-page.html"

[extra]
toc = true

[taxonomies]
categories = ["making Blog"]
tags = ["zola", "github actions"]
+++

zola 를 이용해 만든 블로그를 localhost 에서 github-Pages 로 이동하는 과정입니다.  
zola 의 [Documention](https://www.getzola.org/documentation/deployment/github-pages/)을 따라하면서 놓친 부분을 추가해서 메모해둡니다.

<!-- more -->

## Github Actions
Github-Pages에 Zola-Page를 배포하기 위해 Github Actions를 사용하는 것은 매우 쉽습니다.  
기본적으로 세 가지가 필요합니다.

1. Personal access token 이 없다면 생성
2. Github Action 만들기
3. 리포지토리 설정에서 Github 페이지 섹션을 확인  
   
아래와 같이 순서대로 하면 됩니다.

### 1. Personal access token
동일한 리포지토리에 사이트를 게시하는 경우 해당 단계를 따를 필요가 없습니다.  
그러나 여전히 GITHUB_TOKEN 이 자동으로 전달되게 해야 합니다. 

#### - 토큰을 생성합니다.
[여기](https://github.com/settings/tokens/new?scopes=public_repo)를 클릭 하거나,  
github 페이지 오른쪽 상단 아바타 > Settings > Developer Settings > Personal access tokens > Tokens(classic) > Generate new Token(classic) 으로 이동합니다.  
이름과 유효기간을 정하고, Select scopes 에서 public_repo 권한을 부여합니다.  
Generate token 을 클릭하면 token 이 나오는데, 잘 복사해둡니다.  

#### - 토큰을 환경변수에 저장
리포지토리로 이동하여 상단의 Settings > Environments > New Environment 를 클릭합니다.  
environment 이름을 적은 다음 나오는 페이지에서 Environment secrets > add secret 를 클릭합니다.  
Name 은 TOKEN, Secret 에는 위에서 생성한 token값을 붙여넣습니다.

<br/>

### 2. Github Action 만들기

#### - 만들기
리포지토리 상단의 Actions > New workflow > set up a workflow yourself → 를 클릭합니다.  
아래의 내용을 붙여넣습니다.

```
name: Zola on GitHub Pages

on: 
 push:
  branches:
   - main

jobs:
  build:
    name: Publish site
    runs-on: ubuntu-latest
    steps:
    - name: Checkout main
      uses: actions/checkout@v3.0.0
    - name: Build and deploy
      uses: shalzz/zola-deploy-action@v0.17.2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
작업은 선택한 분기에 대해서만 실행되므로, branch 이름이 main 이 아니라면 master 또는 다른 이름으로 수정합니다.

#### - 권한수정
리포지토리 상단의 setting > actions > general 을 클릭합니다.  
아래쪽의 workflow permission 에서 Read and write permission 선택합니다.

<br/>

### 3. Github 페이지 섹션을 확인  
리포지토리 상단의 setting > Pages 를 클릭합니다.  
Build and deployment 에서 Branch 가 gh-pages 로 설정되어 있고, 디렉토리가 /(root) 인지 확인합니다. 



