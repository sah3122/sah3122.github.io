---
title: Azure API Management Service 활용법
description: Azure API Management Service 활용법
categories:
 - dev
tags:
 - azure
 - cloud
---
> Azure API Management Service를 사용한 API 응답 캐싱

Azure API Management Service 캐싱을 활용한 글로벌 서비스 성능 최적화 방법을 간단하게 공유드립니다. 
해당 방식은 정적인 응답을 반환하는 경우에 사용할 수 있으며 실시간으로 응답이 변경되거나 조건에 따라 다른 응답을 반환하는 경우에는 목적에 맞지 않을수 있습니다.  

## API Management Service란 ?
APIM(API Management)은 기존 백 엔드 서비스를 위해 일관된 최신 API 게이트웨이를 빠르게 만드는 방법입니다.
API Management를 사용하여 조직은 외부, 파트너 및 내부의 개발자에게 API를 게시하여 데이터 및 서비스의 잠재성을 활용할 수 있습니다. 모든 곳의 비즈니스는 디지털 플랫폼으로 운영을 확장함으로써 새로운 채널을 생성하고, 새로운 고객을 찾고, 기존 고객과 더 깊은 관계를 구축하고자 합니다. API Management는 개발자 참여, 비즈니스 통찰력, 분석, 보안과 보호 등을 통해 성공적인 API 프로그램을 보장하는 핵심적인 역량을 제공합니다. Azure API Management를 통해 원하는 백 엔드를 사용하고 해당 백 엔드에 따라 모든 기능을 갖춘 API 프로그램을 시작할 수 있습니다.

**API 게이트웨이** 는 다음 작업을 수행하는 엔드포인트입니다.
* **API 호출 수락 후 백 엔드로 라우팅합니다.**
* API 키, JWT 토큰, 인증서 및 기타 자격 증명을 확인합니다.
* 사용 할당량 및 속도 제한을 적용합니다.
* 코드 수정 없이 즉석에서 API를 변환합니다.
* **설정된 위치에 백 엔드 응답을 캐시합니다.**
* 분석용으로 호출 메타데이터를 기록합니다.

공식 문서에 따르면 위와 같은 기능들을 제공하는 서비스라고 설명되어 있습니다. 
위 기능을 포함하여 많은 기능들을 제공하니 공식 문서를 확인해주시길 바랍니다. 

`API 호출 수락 후 백 엔드로 라우팅, 설정된 위치에 백 엔드 응답을 캐시` 두가지 기능을 사용하여 기존에 사용되고 있던 API를 글로벌 환경에서 효율적으로 서비스 하기 위한 작업을 진행 하였습니다. 

## 사용 배경
현재 사용하고 있는 API서버는 홍콩 리전에 배포되어 있으며 미국 버지니아 기준 간단한 GET 요청을 보낼 시 약 1초 정도의 응답시간이 소요되고 있었습니다.   
트래픽이 많은 서비스는 아니지만 페이지 접근 시 1초 이상의 응답시간이 걸리는 API를 개선하고 준비하고 있는 이벤트 서비스를 사용자 친화적으로 제공하기 위하여 응답시간을 줄이기 위한 작업을 시작하였습니다. 

## 설정
1. API Management 서비스를 생성합니다.  
이때 주의 할 점은 다수의 Location 설정을 위하여 프리미엄 이상의 서비스를 선택해야 합니다. 
![서비스 생성](/assets/images/post/azure-api-management-service/azure-api-management-service-1.png)

2. API 연동
API Management 서비스가 활성화 되면 연동할 API의 정보를 추가해줘야 합니다.  
OpenAPI 등록서비스를 사용하여 추가할 수 있습니다.

![API 연동](/assets/images/post/azure-api-management-service/azure-api-management-service-2.png)
OpenAPI Spec에 맞춘 json파일을 업로드 하여 API Gateway와 연동합니다.  
json파일은 Swagger2.0 기반 api 문서를 그대로 사용할 수 있게 되어있습니다. 

![API 결과](/assets/images/post/azure-api-management-service/azure-api-management-service-3.png)

정상적으로 API연동을 마치면 위와 같이 API목록을 확인 할 수 있습니다. 

3. 캐시 설정 및 지역 추가
API 요청시 응답값을 캐싱하여 빠르게 응답을 전달해주기 위하여 적절한 캐시 설정이 필요합니다. 
공식 문서에서 제공하고 있는 캐시 설정을 적용해보았습니다.  
[Azure API Management에서 캐싱을 추가하여 성능 향상 | Microsoft Docs](https://docs.microsoft.com/ko-kr/azure/api-management/api-management-howto-cache) 
![캐싱 전략](/assets/images/post/azure-api-management-service/azure-api-management-service-4.png)

문서를 참고하여 미국 지역에 새로운 API GW를 추가를 끝으로 캐싱을 위한 설정은 끝이 납니다.  
[Azure API Management 서비스를 여러 Azure 지역에 배포 - Azure API Management | Microsoft Docs](https://docs.microsoft.com/ko-kr/azure/api-management/api-management-howto-deploy-multi-region)

## 테스트
API GW와 캐싱을 사용하여 어느 정도의 성능 향상이 있었는지 정리한 결과입니다. 
테스트는 AWS EC2(버지니아)에서 진행하였습니다.

### 기존 API 요청 응답시간
* 0.961115
* 0.929669 
* 0.920897
* 0.912583
* 0.921966

### API GW 요청 응답시간
* 0.058033
* 0.059246
* 0.065591
* 0.057714
* 0.057355

결과와 같이 대략 15배 이상 응답시간이 단축된것을 확인할 수 있습니다.  

위 설정만으로 완전한 최적화가 이루어지지는 않겠지만 글로벌 서비스를 운영하는 경우 API GW를 사용한 캐싱은
성능 최적화를 위하여 고려해볼 가치가 있는것 같습니다. 
