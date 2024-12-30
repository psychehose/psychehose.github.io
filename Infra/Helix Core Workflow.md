
Update: 2024/12/18

## 목차

1. 소개
2. Stream Depot 구조 이해하기
3. 일반적인 작업 흐름
4. 코드 리뷰 프로세스
5. 주의사항
   
## 소개

이 문서는 우리 팀의 Helix Core 작업 환경과 프로세스를 설명합니다. 개발팀과 아트팀이 효율적으로 협업할 수 있도록 설계되었습니다.

##  Stream Depot 구조 이해하기

우리는 Local Depot 대신 Stream Depot을 사용합니다. Stream Depot의 주요 장점은:

- 브랜치 관리의 용이성
- 변경사항의 명확한 추적
- 팀 간 협업 효율성 향상
- 자동화된 머지 충돌 감지

## Stream 구조 설명

- **Mainline Stream**: 제품의 안정된 버전을 포함하는 최상위 스트림
  
- **Development Stream**: 활발한 개발이 이루어지는 주요 통합 스트림
  
- **Task/Feature Stream**
  - 개별 개발자나 아티스트의 작업 공간
  - Development 스트림에서 분기
  - 특정 기능이나 작업을 위한 독립된 환경
    
- **Release Stream**: 출시된 버전을 보관하는 스트림

## 일반적인 작업 흐름

1. 작업 시작하기
   - Development 스트림에서 새로운 Task/Feature 스트림 생성
   - 작업할 스트림으로 전환
     
2. 변경사항 제출
   - 자신의 Task/Feature 스트림에는 자유롭게 submit 가능
   - Development 스트림으로의 머지는 코드 리뷰 필수
     
3. 머지 프로세스
   - 작업 완료 후 Development 스트림과 머지 진행
   - Swarm을 통한 코드 리뷰 승인 필요
   - 승인 후 머지 진행

## 코드 리뷰 프로세스

Swarm 설정이 필요한 부분:

-  리뷰어 자동 할당 규칙 설정
-  승인 필요 인원 수 설정
-  머지 전 필수 리뷰 정책 설정
-  리뷰 만료 기간 설정
-  자동 알림 설정

1. 스트림 접근 제한
   - Development와 Mainline 스트림에 직접 submit 금지
   - 반드시 머지 프로세스를 통해서만 변경사항 반영

2. 코드 품질 관리
   - Mainline 스트림에는 검증된 코드만 허용
   - 모든 머지는 코드 리뷰 승인 필수

