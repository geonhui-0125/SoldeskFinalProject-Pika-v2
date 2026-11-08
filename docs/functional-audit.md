# 원본 기능 대조 기록

- 검토일: 2026-11-08
- 기준: 저장소의 원본 기획서와 현재 소스 정적 검토. 런타임 전체 검증 결과가 아니다.
- 목적: 원본의 계획·코드 존재·실제 규칙을 구분하고 v2 재설계의 근거를 남긴다.

| 영역 | 확인한 구현·계획의 차이 | v2에서 재정의할 사항 | 근거 |
| --- | --- | --- | --- |
| 인증 | 기획서는 JWT를 언급하나 현재는 폼·세션 로그인. JWT는 미사용 | JWT 수명·로그아웃·갱신과 실제 요청별 권한 | [LoginSecurity](../pika/src/main/java/com/numlock/pika/config/LoginSecurity.java) |
| 접근 제어 | HTTP 필터는 전체 경로 허용. 일부 메서드·컨트롤러가 별도로 인증·권한 검사 | 전체 허용 설정과 개별 검사를 대체하는 일관된 권한표 | [LoginSecurity](../pika/src/main/java/com/numlock/pika/config/LoginSecurity.java), [ProductController](../pika/src/main/java/com/numlock/pika/controller/ProductController.java) |
| 결제 검증 | PG 응답 존재 여부를 검사하나 금액 비교는 요청값과 DB 상품 가격 사이에서 수행 | PG의 실제 금액·상태·주문·구매자 검증과 멱등성 | [PaymentApiService](../pika/src/main/java/com/numlock/pika/service/payment/PaymentApiService.java) |
| 결제 취소 | 취소 메서드는 상품 상태 복구와 로컬 결제 레코드 삭제를 수행. 해당 메서드에 PG 환불 호출 없음 | 환불 결과 확인·이력 보존·재판매 조건 | [PaymentApiService](../pika/src/main/java/com/numlock/pika/service/payment/PaymentApiService.java) |
| 거래 상태 | 상품 필드에 0·1·2·3 상태를 기록. 엔티티 설명은 판매 중·완료만 정의 | 상품·주문·결제 상태 분리와 명령 주체·전이 정의 | [Products](../pika/src/main/java/com/numlock/pika/domain/Products.java), [PaymentApiService](../pika/src/main/java/com/numlock/pika/service/payment/PaymentApiService.java) |
| 알림 | 구매 확정 컨트롤러는 서비스의 확정 처리 전에 알림 호출 | 업무 처리 성공 후 재처리 가능한 알림 발생 | [PaymentApiController](../pika/src/main/java/com/numlock/pika/controller/payment/PaymentApiController.java) |
| 리뷰 | 기획은 구매 확정 후 작성. 서비스는 구매자·상품에 대한 결제 이력 존재를 확인하고 중복·자기 리뷰를 검사 | 구매 확정 주문 기준 자격과 주문당 유일성, 작성자·판매자 정보의 서버 도출 | [ReviewServiceImpl](../pika/src/main/java/com/numlock/pika/service/ReviewServiceImpl.java) |
| 메시지 | 저장 전에 발신자에게 반환. 발신자를 payload에서 가져오고 수신자 접속 여부로 읽음 값을 결정 | 인증된 발신자·방 권한, 저장 ACK, 전달·읽음 구분 | [WebSockChatHandler](../pika/src/main/java/com/numlock/pika/handler/WebSockChatHandler.java) |
| 접속 관리 | 사용자 ID당 한 WebSocket 세션을 프로세스 메모리에 저장 | 다중 탭·기기·재접속·버퍼 한도 | [WebSockChatHandler](../pika/src/main/java/com/numlock/pika/handler/WebSockChatHandler.java) |
| 관리자 | 제재·해제·상품 위반·관리 삭제 엔드포인트는 있으나 해당 상태 변경 코드가 주석 처리 | 조회와 실제 제재 기능 구분, 상태·사유·감사 기록 | [AdminUserServiceImpl](../pika/src/main/java/com/numlock/pika/service/AdminUserServiceImpl.java), [AdminProductServiceImpl](../pika/src/main/java/com/numlock/pika/service/AdminProductServiceImpl.java) |
| AI | HTTP 요청으로 완성 문자열 응답, 세션 ID별 이력은 JVM 메모리. 상품 Repository와 모델 호출이 같은 서비스에 결합 | 회원 소유 대화·스트리밍·취소·영구 이력·검색/생성 경계 | [AiChatController](../pika/src/main/java/com/numlock/pika/controller/AiChatController.java), [GeminiService](../pika/src/main/java/com/numlock/pika/service/GeminiService.java) |

[기능명세](functional-spec.md)의 수락 기준은 위 차이를 포함해 설계한다. 기존 API가 있다는 사실만으로 기능 완료를 판단하지 않는다.
