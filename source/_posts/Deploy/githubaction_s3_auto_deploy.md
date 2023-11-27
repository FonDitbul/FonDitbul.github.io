---
title: Github Action 과 S3로 웹 페이지 자동 배포하기
toc: true
published: true
date: 2023-11-27 22:16:03
tags:
  - github action
  - aws s3
categories:
  - Deploy
  - Front
---
> Github Action 과 S3의 정적 호스팅 기능을 사용하여 **코드가 push 될 때마다 웹 페이지를 자동 배포**할 수 있는 방법을 알아보도록 하겠습니다.

# AWS IAM 만들기
* 먼저 IAM 을 통해 Github Action에서 사용할 계정을 만들어 보도록 하겠습니다.

## IAM 계정 생성하기
{% asset_img IAM_create.PNG %}
* AWS 콘솔에서 IAM검색 후 사용자 > 사용자 생성 클릭

## IAM S3 권한 설정하기
{% asset_img IAM_2_auth.PNG %}
* 권한 정책에서 AmazonS3fullAccess 를 선택하여 S3에 대한 모든 권한을 부여하는 계정을 생성합니다.

## IAM key 발급받기 
{% asset_img IAM_access_key.PNG %}
* 여기서 access key와 private key 를 복사해서 하단의 github 에 붙여넣기 합니다.


# Github 설정하기
* Github Repository 에서 Setting 을 다음과 같이 설정하기
{% asset_img github_settings.PNG %}
* ```Settings``` 하단에 ```Secrets and variables``` > ```Actions``` 를 클릭
* Repository secrets 에서 ```New repository secret```를 클릭 
* IAM key 에서 받아온 값들을 그림과 같이 저장합니다.


# S3 버킷 정적 호스팅 설정하기
## 1. s3 bucket 만들기
* aws 콘솔에서 s3 검색
* ```버킷 만들기 클릭```

{% asset_img bucket_create.PNG %}

## 2. s3 bucket setting 엑세스 차단 설정 해제하기
{% asset_img s3_setting.PNG %}
* 해당 s3 버킷은 외부에서 접근을 해야하기 때문에 위와 같이 엑세스 차단 설정을 해제 해야 합니다.

## 3. s3 버킷 설정하기
{% asset_img bucket_setting.PNG %}
* 생성된 버킷 클릭 > 속성 탭 클릭 > 하단에 ```정적 웹 사이트 호스팅``` 에서 편집 버튼을 클릭
* 인덱스 문서에 기본 페이지 파일을 입력합니다.
  * 보통 ```index.html```을 사용합니다.

{% asset_img s3_static_site_setting.PNG %}

## 4. s3 정책 다음과 같이 설정하기
* 버킷 > 권한 탭 클릭 > ```버킷 정책```에서 편집 클릭 후 아래와 같이 작성 및 적용
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::github-action.test/*"
    }
  ]
}
```
* ```github-action.test``` 은 설정한 s3 버킷 이름을 작성하여야 합니다.


# Github Actions yml 생성하기
* 호스팅 하고자 하는 Github Repository 에서 다음과 같은 파일을 생성합니다.
* 파일 이름은 변경해도 상관없으나 경로는 유지해야합니다.
```yml
# .github/workflows/deploy.yml
name: Front Deploy

on:
  push:
    branches: ['main']

jobs:
  deploy:
    runs-on: ubuntu-latest
#    defaults:
#      run:
#        working-directory: ./
    steps:
      - name: Checkout source code
        uses: actions/checkout@master

      - name: Cache node modules # node modules 캐싱
        uses: actions/cache@v1
        with:
          path: node_modules
          key: ${{ runner.OS }}-master-build-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.OS }}-build-
            ${{ runner.OS }}-

      - name: Install
        run: npm i

      - name: Build
        env:
          CI: false
        run: npm run build

      - name: Deploy
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 cp \
            --recursive \
            --region ap-northeast-2 \
            build s3://github-action.test
```

* 상단의 on push branches 에 의해 ```main 브랜치에 push가 될때마다 해당 Github Action이 실행```됩니다.

* 이후 해당 repository 에 actions 탭에서 제대로 완료되었는지 확인

{% asset_img github_action_complete.PNG %}

### 끝
* s3에 파일 업로드 확인 
* 마지막 정적 웹사이트 호스팅 하단에 링크 클릭하면 제대로 출력되는지 확인!
{% asset_img static_hosting_complete.PNG %}






