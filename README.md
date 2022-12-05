![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 예제 - 음식배달

본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다.
이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW


# 서비스 시나리오

기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다
1. 고객이 선택한 메뉴에 대해 결제한다
1. 결제가 완료되면 주문 내역이 입점상점주인에게 주문정보가 전달된다
1. 상점주인이 주문을 수락하거나 거절할 수 있다
1. 상점주인은 요리시작때와 완료 시점에 시스템에 상태를 입력한다
1. 고객은 아직 요리가 시작되지 않은 주문을 취소할 수 있다
1. 요리가 완료되면 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다
1. 고객이 주문상태를 중간중간 조회한다
1. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
1. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다.


비기능적 요구사항 
1. 장애격리
    1. 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다  Circuit breaker, fallback
1. 성능
    1. 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다  CQRS
    1. 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다  Event driven


# 체크포인트

## Event Storming 결과
* MSAEz 로 모델링한 이벤트스토밍 결과
![image](https://user-images.githubusercontent.com/43160394/205589842-d5311b7e-fffb-461b-8911-fbf5cbdbf337.png)

* Github link : 
## 1. Saga (Pub/Sub)




실행 결과


## 2. CQRS
Read Model CRUD 상세설계
![image](https://user-images.githubusercontent.com/43160394/205553678-3735a513-4cc5-43b9-8ca9-b69be33d0e1b.png)

실행 결과


## 3. Compensation/Correlation


실행 결과


## 4. Request/Response


실행 결과


## 5. Circuit Breaker
호출선의 설정에서 Circuit breaker 옵션을 On 한다
![image](https://user-images.githubusercontent.com/43160394/205560198-bd1d95b2-95b1-422b-8a2c-8064beb63b2c.png)

생성 코드 확인과 구현
서킷브레이커 설정
front 서비스의 application.yaml 파일의 다음 설정을 true 로 하고, 임계치를 610ms으로 변경
![image](https://user-images.githubusercontent.com/43160394/205564504-8c817d83-b177-4595-aac2-0762dc4f7f77.png)


실행 결과


## 6. Gateway/Ingress


실행 결과




