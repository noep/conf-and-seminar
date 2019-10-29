# 2019-deview-day2

## TRACK1 네이버 로그를 지탱하는 힘 (DataStore 로그 저장소)
이윤경 / 강민우
https://deview.kr/data/deview/2019/presentation/[211]%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A1%9C%EA%B7%B8%EB%A5%BC+%EC%A7%80%ED%83%B1%ED%95%98%EB%8A%94+%ED%9E%98_20191025_%EA%B0%95%EB%AF%BC%EC%9A%B0_%EC%9D%B4%EC%9C%A4%EA%B2%BD.pdf

### 로그 저장 
유실 없는 로그 저장 
- at least once
- 데이터 중복이 발생할 수 있음
- 중복 처리 : 고유 식별자를 이용한 idempotent 쓰기
- 고유 식별자 부여가 어려움
 - 방법 1 해싱
  - 그래도 겹침
 - 방법 2 채번서버 두기
  - SPOF가 될 수 있음
 - 방법 3 UUID 사용
  - 구현 방법에 따라 다르나 어느정도 표준
  - 버전1을 모방하여 사용
   - mac주소 + 시간 --> 모방 --> sender server ip + 시간 + variant 값(사용자 입력)(로그 파인 라인 위치 등)

- RowKey 디자인
 - Salt // 리전(R) / 그룹(G) / 메세지 타입(M) / 로그생성 시간 / IP / Variant
 - RGM
  - R : Korea / G : naver_search / M : query_log
 - Hbase Table의 경우 시간 순서대로 연속 읽기 가능
 - 그러나 시간값 같은 hotSpot 발생 가능
 - -> Unique Key 이용해서 Salt 사용
 - 저장 공간을 용도별로 나눔

 - Data Catalog
  - 사용자 역할 관리 
  - 데이터 필드 정보 및 포맷 관리
 - 티켓
  - 리소스 관리 및 접근 제어
  - 용도별로 발급 / 유량제어

### 로그 활용 이야기
- 기술 : HIVE가 핵심
- 네이버 검색 로그
 - 새로운 서비스의 시작
 - 사용자 반응의 바로미터
 - 하루에 수십억 건 / 수 TB
 - 높으신 분들이 찾음
  - 유형도 다양 / 분석 의뢰도 수백건
  - 분석자 : 분석 의뢰 프로세스도 비효율적임 결재 / 수동적 / 폐쇄적
  - 작업자 : 매번 비슷한 패턴의 추출 로직 / 반복적인 로직 / 서비스에 대한 이해 부족
  - 의뢰자 분석자 모두 불편한 상황
  - -> 로그를 오픈하고 자유롭게 분석하기
   - 보안 이슈 / 사용권한 있어야 함
   - 데이타 스토어 도입
   - 생산자 : 안전하게 위탁
   - 사용자 : 필요한 만큼(권한/보안)만 사용
  - 로그 사용의 어려움
   - 비개발자가) 분산파일 시스템을 알아야 하고, 프로그래밍 해야함, 터미널 익숙, 맵리듀스 짜야함
   - -> SQL Interface 필요 / HIVE 도입
   - SQL Tool
    - beeline
    - hue
    - zeppelin
    - ...
   - 그래도 복잡함
   - *가공되지 않은 로그에서 비즈니스적으로 의미있는 지표를 추출하는 것은 또 다른 문제!!*
    - 그간 쌓아둔 로그 분석 의뢰건을 전수 조사하여 유형별로 정리 -> 템플릿 화
    - Hue를 이용해 문서화 (템플릿 기능 사용)
    - 템플릿으로 해결되지 않는 분석 의뢰건
     - SQL Archive 이용 (github)
     - 이슈 등록 -> github에 sql 등록
     - SQL Template과 SQL Archive를 이용하여 유사 분석 의뢰를 찾을 수 있게 함
     - 맵리듀스를 내가 해줌 -> (권한을 줄 테니) 쿼리를 너희가 짜서 써
#### 이게 최선인가? SQL 도 복잡하다
- 필요한 내용에 대해서 가공 테이블을 미리 만들어 제공 (조인 작업 등)
- HBase -> ORC Format으로 저장한 것이 도움이 되었다

#### 가공 테이블의 성능 유지를 위해 파티션과 Bucket을 사용한다.
- 테이블 풀 스캔 지양 
- Partition : 날짜 / Bucket : 검색어
- 특정 검색어로 인한 hotspot 발생 가능
#### External Table 이용
- 안전 / 데이터 소스 변경 용이 / hdfs가 아니어도 테이블화 가능
- HIVE Transaction이나 LLAP 사용시 제약이 있음
#### Hive UDF
 - 검색 로그 특화된 UDF 제공

#### HIVE 도입은 작년부터!!
- hive 2.x view 안정적이지 않음

### QNA
- RGM중 M은 어떻게 카테고라이즈 하는가?
 - 사용자가 정함
- Hbase로 로그를 먼저 받는 이유와 장점
 - ORC가 HBASE에 비해 성능이 훨씬 좋음 처음부터 ORC쓰는게 낫지 않냐 라는 질문도 받음
 - HBASE로 받는 이유
  - 처음 hdfs -> 사용자 많아짐 / 안정성 떨어짐 -> hbase로 전환


## TRACK1 Pinpoint는 어떻게 observability를 강화했는가
구태진


## TRACK1 ms 단위의 Serverless World에서 Docker의 성능 한계 극복하기
김동경
## TRACK2 Fail Fast, Learn Faster SRE (실패에서 배워나가는 SRE)
김재헌 / 유호균



## TRACK2 React, Angular, Vue를 한 번에 지원하기 위한 설계 (Cross Framework Component)
최연규

## TRACK2 2019년 FE 프레임워크를 배우는 기분(FE 인싸들이라면 알고 있어야 하는 프레임워크 기술들)


## TRACK3 Armeria: 어디서나 잘 어울리는 마이크로서비스 프레임워크