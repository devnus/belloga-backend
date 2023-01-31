
<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215764993-82b51eaf-9aa8-434e-ae66-b88d9975d049.png"></p>

# 프로젝트 개요

## 1-1. 프로젝트 소개

벨로가: 기상알람을 활용한 라벨링 데이터 수집 모바일 애플리케이션

데이터 라벨링을 사용자가 지속적으로 참여하게끔 하기 위해 게임화 요소로 기상 알람에 OCR 데이터 라벨링을 수행할 수 있도록 하고 라벨링에 대한 보상으로 포인트를 지급후 해당 포인트는 기프티콘 이벤트 응모에 사용 가능합니다.

## 1-2. 주요 기능 통신 다이어그램

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215765572-39c8d96b-9a00-41ac-9a5b-5a487e624897.png" width="680" height="850"></p>

## 1-3. 주요 기능

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215766327-52299838-9442-4012-b985-85f097934f21.png"></p>

## 1-4. 개발 환경

- Front-end: `React Native` `Android` `React`
- Back-end: `Java` `Python` `Spring Boot` `JPA` `MySQL` `Apache Kafka`
- DevOps: `Docker` `AWS` `Kubernetes` `istio` `Jenkins` `zipkin`

# 개발 결과물

## 2-2. AWS 아키텍처

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215766497-7ca32a36-c2e5-4883-9ab4-559bc3a1101f.png"></p>

쿠버네티스를 기반으로 AWS 아키텍처를 설계하고 개발하였습니다. 이 과정에서 쿠버네티스 관리형 서비스인 `EKS`를 사용하였습니다.
먼저 마이크로서비스들의 인증 인가를 통합적으로 수행하기 위해 앞단에 `API Gateway`를 두었습니다. 또한 마이크로서비스 간 비동기 통신을 위해 `Apache Kafka` 관리형 서비스인 `MSK`를 사용하였습니다.
이외에도 `S3`, `ECR`, `ElastiCache` 등 다양한 AWS 관리형 서비스를 활용하였습니다.

## 2-1. DDD 도입

### 2-1-a. 이벤트스토밍을 통한 바운디드 컨텍스트기반 마이크로서비스 도출

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215766660-1a358c5b-f669-4ab2-b59b-6ad61cd7c78c.png"></p>

`DDD`의 이벤트스토밍을 통해 바운디드 컨텍스트 단위로 마이크로서비스를 도출하였습니다. 

- **belloga-auth-service**
→ `스프링`으로 구현, JWT 토큰 기반 인증 인가 담당
- **belloga-user-service**
→ `스프링` 으로 구현, 사용자(기업, 일반, 관리자) 프로필 관리 담당
- **belloga-raw-data-service**
→ `스프링` 으로 구현, raw data 관련 기능 담당 (기업 사용자의 raw data(OCR, 오디오 등) 업로드, 관리자의 raw data 승인 등)
- **belloga-preprocessing**
→ `파이썬` 으로 구현, 관리자가 승인한 raw data에 대해 알맞는 전처리 파이프라인 담당
- **belloga-labeling-service**
→ `스프링` 으로 구현, 데이터 라벨링 수행 담당
- **belloga-labeling-vertification-service**
→ `스프링 배치` 로 구현, 주기적으로 사용자가 수행한 라벨링에 대해 검증 수행 배치 담당
- **belloga-point-service**
→ `스프링` 으로 구현, 사용자 보상(포인트, 스탬프, 이벤트 응모 등) 담당
- **belloga-firebase-service**
→ `스프링` 으로 구현, firebase 관련 비즈니스 응집성 담당(fcm 푸쉬알림)

### 2-1-b. 애그리거트 패턴 적극 사용

- 애그리거트 내 상세 클래스를 바로 참조하지않고 애그리거트 루트만 참조하였습니다.
- 애그리거트 간의 참조는 객체를 직접 참조하는 대신 기본키(id)를 사용하여 수정이 필요하지 않은 애그리거트를 함께 수정하는 실수를 원천적으로 방지하였습니다.

## 2-2. 마이크로서비스 가트너 아키텍처

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215766772-50b29926-58d5-4ccb-9143-0178260a869a.png"></p>

벨로가의 가트너 아키텍처입니다.

클라이언트-마이크로서비스 간 직접 통신 방식이 아닌 `API Gateway` 패턴을 사용하였고 이를 통해 인증 / 인가를 통합적으로 수행하였습니다.

마이크로서비스들 간 통신을 제어하기 위해 서비스 매시 레이어를 넣었고, 이에 대한 솔루션으로 `isito`를 사용하였습니다. `istio` 를 통해 마이크로서비스 간 동기 통신 과정에서 발생할 수 있는 장애 전파문제를 해결하기 위해 `circuit breaker`를 구현하였습니다.

이벤트 주도 아키텍처를 적극 도입하여 동기통신이 필수적으로 필요한 로직을 제외하고 마이크로서비스 간 통신은 비동기로 설계, 개발하였습니다. 이벤트 브로커 솔루션으로 `아파치 카프카`를 사용하였습니다.

## 2-131. 분산 로그 트레이싱 zipkin 도입

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215766924-12de94cc-2b47-4b8d-aac9-5d14ff896b9e.png"></p>

마이크로서비스로 설계, 개발함에 따라 사용자로부터의 요청 흐름 중에서 어느 지점에서 장애나 병목 현상이 발생하였는지 점점 추적하기 어려웠고,`zipkin`을 도입하여 문제를 해결하였습니다. 이 과정에서 `Spring Cloud Sleuth` 를 사용하였습니다.

## 2-10. API Gateway 패턴을 활용한 인증 / 인가 구현

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767058-2e68d272-294b-4092-bd72-2c980914d6d4.png" width="630" height="800"></p>

`JWT`를 이용한 사용자 인증 방식으로 구현을 했는데, 이때 각 마이크로서비스가 토큰의 인증/인가를 중복으로 구현한다면 비효울적이라고 생각했습니다.

이에 대한 해결 방안으로 `API Gateway`를 통한 토큰 검증 패턴을 도입했습니다. 이를 통해 각 마이크로서비스의 API를 이용하기 위해서는 앞단의 `API Gateway`를 거치도록 설계하였고, `API Gateway`에서 `Lambda Authorizer`를 통해 `JWT` 토큰이 검증되고, 검증된 토큰의 페이로드에 저장되어있는 대칭키로 암호화한 사용자 식별값을 다시 복호화하여 뒷단의 마이크로서비스에 전달되도록 구현했습니다. 

## 2-3. 데이터베이스 ERD 및 RDB 구조

- `auth service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767183-e712a9cc-471e-46e6-a922-9faf6885c572.png" width="38%"></p>
    
- `point service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767303-d74fb11a-c95d-4870-ab71-e40d837c43b6.png" width="38%"></p>
    
- `labeling service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767428-9383ad7a-77ba-413a-993a-5671f57df740.png" width="38%"></p>
    
- `user service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767471-93a134ce-8534-45dd-9a02-37dcc031ef99.png" width="38%"></p>
    
- `firebase service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767506-77d8edc0-56ac-4008-9e21-0fb7f6084dff.png" width="38%"></p>
    
- `labeling verification service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767545-84fac0a5-991c-4ac1-b2c2-723fab62155a.png" width="38%"></p>
    
- `raw data service`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767587-53d35044-21d7-4852-ab03-4515c9932ad7.png" width="38%"></p>
    
- `모든 마이크로서비스의 스키마를 합칠경우`

    <p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767684-5faabce5-f8f9-471a-9477-f9ae2381f573.png" width="38%"></p>
    

## 2-4. 마이크로서비스 API 통합 문서화

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767731-c7b80351-2ce3-48d0-98d6-628cf0fbe936.png"></p>

마이크로서비스들의 API를 통합 관리하기 위해, 각 마이크로서비스의 API를 `Spring REST Docs`로 문서화 한뒤, `Swagger UI`를 통해 통합 관리하였습니다.

## 2-7. API 서버 CI/CD 파이프라인

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767766-0f51393b-5889-4a77-9fcf-193156774944.png"></p>

`Jenkins`를 통해 CI/CD 파이프라인을 구축하였습니다. 이때, 각 마이크로서비스 별로 Open API Spec을 추출해 `Swagger UI` 서버로 전송하여 API 문서 통합을 수행하으며, 도커 이미지를 만들어 `ECR`에 업로드 후 `EKS`에서 업로드한 도커 이미지를 받아 Rolling Update를 수행했습니다.

## 2-8. AWS S3 Presigned URL 기반 미디어 업로드

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767827-154daf82-68a8-4c5a-801e-6ba34a966138.png"></p>

서비스 설계시, 라벨링을 의뢰하기 위해 업로드하는 데이터가 서버를 거쳐 `S3` 버킷으로 업로드 되도록 설계하였는데, 라벨링을 의뢰하는 데이터가 벌크데이터라는 점에서 서버에 많은 부하가 발생할것이라고 판단했습니다.

이에 대한 해결 방안으로 서버를 거치지 않고 `S3`에 다이렉트로 업로드할 수 있는 `pre-signed URL`을 도입하였고, 이를 통해 벌크데이터가 서버를 거치지 않고 바로 `S3`버킷으로 업로드 되도록 구현했습니다.

---

<p align="center"><img src="https://user-images.githubusercontent.com/57690870/215767909-7ebf48db-788b-49f3-b3df-38c88176e48e.png" width="38%"></p>

*This Project is Sponsored by **Software Maestro***

This work was supported by the Institute of Information & Communications Technology Planning & Evaluation(IITP) grant funded by the Ministry of Science and ICT(MSIT) (IITP-2022-SW Maestro training course).
