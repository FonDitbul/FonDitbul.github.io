---
title: 
type: about
widgets:
  - type: toc
    position: right
  - position: right
    type: profile
    author: 정권기
    author_title: With Node
    location: Seoul
    avatar: https://avatars.githubusercontent.com/u/49264688?v=4
    avatar_rounded: false
    social_links:
        Blog: https://fonditbul.github.io/
        Github:
            icon: fab fa-github
            url: https://github.com/FonDitbul
toc: true
---
# **정권기**
> 과정을 즐기는 벡엔드 개발자


# 자기소개
> 개발 과정을 즐기며 이야기 하는 것을 좋아하는 2년차 Node 백엔드 개발자 정권기 입니다. 
* 결과보다 과정에 있어 즐거움을 찾습니다.
  * 문제 해결에 대한 이야기를 좋아합니다.
  * 개발에 몰입했을 때 시간이 빨리 지나 있음에 성취감을 느낍니다.
* 변화에 대응이 쉽고 유지보수와 지속 가능한 애플리케이션에 대해 계속 고민합니다.
  * 여러 개발 방법과 스터디를 통한 성장에 욕심이 있습니다. 
* 항상 주어진 일에 책임감 있게 수행하고 더 만족도 높은 결과를 내고자 합니다.
  * 능동적으로 일을 찾고 해결하고자 합니다.


*** 
# 스킬
### Backend
* **Typescript**, **NestJs**, **Express**
* Sequelize, Prisma
* PostgreSQL, MongoDB
* Jest, Sinon 
### Frontend
* HTML, CSS
* React(기초 수준)
### Etc
* AWS Ec2, ALB, S3, RDS
* Docker
* Github
* Webstorm

***
# 경험
### **사이드 프로젝트**
#### 2023.04. ~ 
#### copang
#### [프로젝트 설명]
> 현업에서 발생할 법한 여러가지 이슈들의 해결, 개발 능력 향상을 위한 사이드 프로젝트 입니다.
* 이커머스 사이드 프로젝트
* 아키텍처, 테스트 코드, 리팩터링, 코드 관리 등에 집중하며 개발 

#### [개발]
* Typescript, Nest, PostgreSQL, Prisma 스택 사용
* 외부 의존성 변화에 따른 개발을 최소화 하기 위한 Hexagonal 아키텍처 를 참고하여 구성

#### [인프라]
* 코드 관리 및 프로젝트 관리를 위한 모노레포 조직
* git flow 브랜치 전략을 사용한 관리 
* [[Copang Repository]](https://github.com/FonDitbul/copang_2_0)


### **인졀미** 
#### 2022.03. ~ 2023.06.
[프로젝트 설명]
> [우주두잇] 프로젝트는 아동청소년을 대상으로 한 게임형 건강관리 헬스케어 서비스이며 전국 52개 초등학교에서 3000여명의 사용자에게 서비스를 제공하는 업무를 담당하였습니다.
* 우주두잇 프로젝트 Backend 개발

##### [API 및 코드]
* 코드 컨벤션을 위한 eslint, prettier 를 통한 formatting 적용
* Validation 코드의 관심사 분리와 불필요한 중복코드 제거를 위한 Middleware 활용
* 견고하며 확장성 있는 서버 코드를 위한 단위 테스트 코드 도입 및 약 2달간 테스트 케이스 234개 작성

##### [데이터베이스 개선]
* RDB 쿼리 실행 계획 및 인덱스를 활용한 성능 튜닝 작업 진행 메인 페이지 API 중 쿼리 속도 99% 개선
* MongoDB 인덱스를 활용한 건강 데이터 중복 데이터 제거 속도 97% 개선

##### [인프라]
* APM Sentry On Premise 도입
* 기존 수동 배포 절차를 쉘 스크립트를 활용하여 자동화하여 배포시간 83% 감소
* 용량 부족시 수동으로 진행했던 서버 건강 데이터 s3에 자동 업로드 개선
* AWS ec2, ALB를 활용한 365/24 서비스 제공

### **과제전형 대비 E-Commerce 마이크로 서비스 설계**
#### 2023.04.17 ~ 2023.04.30
##### [프로젝트 설명]
> **[Numble]** 에서 주관하는 [과제전형 대비 E-Commerce 마이크로 서비스 설계] 과제
* E-commerce 상에서 쿠폰 기능을 도입하기 위한 가상의 환경을 통해 요구사항을 만족하는 범용적인 쿠폰(할인) 마이크로 서비스를 만드는 프로젝트 진행
* [Numble E-commerce 마이크로 서비스 설계 가이드라인 링크](https://www.notion.so/E-commerce-7b52226463074ef08430aaf7ba8fc3d0)  

##### [개발]
* Nest Js, Typescript, PostgreSQL, Prisma(ORM), gRPC의 기술 스택으로 개발
* Redis 를 활용하여 쿠폰 동시 발급 이슈 해결
* k6를 활용하여 성능 테스트 진행
* 지표 시각화를 위한 grafana 사용

##### [결과]
* 참가자 9명 중 TOP-1 달성
* 프로젝트 종료 후 Redis를 활용하여 150 RPS -> 800 RPS 성능 개선

* [[Numble-Coupon Repository]](https://github.com/FonDitbul/numble-coupon)


### **GameLog-Server**
#### 2021.08. ~ 2021.11.

#### [프로젝트 설명]

> 대학교 캡스톤 디자인 으로 진행된 사용자 기반의 게임 평가, 게임 일기, 게임 추천 서비스 제공하는 앱 서비스 GameLog 개발

##### [개발]
* Javascript, express, mopngodb MVC 패턴에 따른 구현
* [[GameLog-Server Repository]](https://github.com/SultanOfGamer/GameLog-Server)



### **딥노이드**
#### 2020.09 ~ 2021.02.
#### [프로젝트 설명]
> 공항 X-ray를 통한 위해 물품 탐지 업무 보조 AI 위해물품 자동 탐지 어플리케이션 개발 프로젝트를 진행하였습니다.

#### [개발]
* 레이블링을 편의성을 위한 내부 레이블링 사이트 유지보수 및 위해 물품 자동 탐지 어플리케이션 프론트엔드 개발

***
# 경력
### 인졀미 
Node 백엔드 개발자 
#### 2022.03. ~ 2023.06. (1년 4개월)
* 헬스케업 플랫폼 스타트업
* [인졀미 회사 링크](https://www.injewelme.com/index.html)

### 딥노이드
프론트 엔드 개발자
#### 2020.09. ~ 2021.02. (6개월)
* 산업, 의료 AI 개발 관련 기업
* [딥노이드 회사 링크](https://www.deepnoid.com/)

***
# 학력
#### **광영고등학교** 이과계열
2012.03. ~ 2015.02. | 졸업

#### **서울과학기술대학교**
대학교(학사) | 컴퓨터공학과  
2015.03. ~ 2022.02. | 졸업

*** 
# 자격증
#### **정보처리기사**

2021.11.


