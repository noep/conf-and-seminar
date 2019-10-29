# 2019-deview-day2

## TRACK1 네이버 로그를 지탱하는 힘 (DataStore 로그 저장소)
이윤경님 / 강민우님

[발표자료](https://deview.kr/data/deview/2019/presentation/[211]%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A1%9C%EA%B7%B8%EB%A5%BC+%EC%A7%80%ED%83%B1%ED%95%98%EB%8A%94+%ED%9E%98_20191025_%EA%B0%95%EB%AF%BC%EC%9A%B0_%EC%9D%B4%EC%9C%A4%EA%B2%BD.pdf)

### 로그 저장 
#### 유실 없는 로그 저장 
- at least once
  - 데이터 중복이 발생할 수 있음
  - 중복된 데이터 처리 : 고유 식별자를 이용한 idempotent 쓰기
- 고유 식별자 부여가 어려움
  - 방법 1 해싱
    - 그래도 겹침
  - 방법 2 채번서버 두기
    - SPOF가 될 수 있음
   - 방법 3 UUID 사용
      - 구현하는 방법이 여러가지 존재
      - 버전1을 모방하여 사용
      - mac주소 + 시간 --> 모방 --> sender server ip + 시간 + variant 값(사용자 입력)(로그 파인 라인 위치 등)
- RowKey 디자인
  - Salt // 리전(R) / 그룹(G) / 메세지 타입(M) / 로그생성 시간 / IP / Variant
  - RGM
    - R : Korea / G : naver_search / M : query_log
  - Hbase Table의 경우 시간 순서대로 연속 읽기 가능
  - 그러나 시간값 같은 hotSpot 발생 가능 -> Unique Key 이용해서 Salt 사용
  - 저장 공간을 용도별로 나눔

#### Data Catalog
- 사용자 역할 관리 
- 데이터 필드 정보 및 포맷 관리
#### 티켓
- 리소스 관리 및 접근 제어
- 용도별로 발급 / 유량제어

### 로그 활용 이야기
- 기술 : HIVE가 핵심
#### 네이버 검색 로그의 특징
  - 새로운 서비스의 시작
  - 사용자 반응의 바로미터
  - 하루에 수십억 건 / 수 TB
  - 높으신 분들이 찾음
    - 유형도 다양 / 분석 의뢰도 수백건
  - 분석자 : 분석 의뢰 프로세스도 비효율적임 결재 / 수동적 / 폐쇄적
  - 작업자 : 매번 비슷한 패턴의 추출 로직 / 반복적인 로직 / 서비스에 대한 이해 부족
  - 의뢰자 분석자 모두 불편한 상황 -> 로그를 오픈하고 자유롭게 분석하자
    - 보안 이슈 / 사용권한 있어야 함
    - 데이터 스토어 도입
    - 생산자 : 안전하게 위탁
    - 사용자 : 필요한 만큼(권한/보안)만 사용
  - 로그 사용의 어려움
    - 비개발자가) 분산파일 시스템을 알아야 하고, 프로그래밍 해야함, 터미널 익숙, 맵리듀스 짜야함
    - -> SQL Interface 필요 / HIVE 도입
    - SQL Tool
      - beeline, hue, zeppelin, ...
    - SQL 툴을 도입해도, 그래도 복잡함
      - 템플릿 이용
        - *가공되지 않은 로그에서 비즈니스적으로 의미있는 지표를 추출하는 것은 또 다른 문제!!*
        - 그간 쌓아둔 로그 분석 의뢰건을 전수 조사하여 유형별로 정리 -> 템플릿 화
        - Hue를 이용해 문서화 (템플릿 기능 사용)
      - 템플릿으로 해결되지 않는 분석 의뢰건 : SQL Archive 운영
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

#### HIVE 도입은 작년부터 진행함
- hive 2.x view 안정적이지 않음

### QNA
- RGM중 M은 어떻게 카테고라이즈 하는가?
 - 사용자가 정함
- Hbase로 로그를 먼저 받는 이유와 장점
  - ORC가 HBASE에 비해 성능이 훨씬 좋음 (처음부터 ORC쓰는게 낫지 않냐 라는 질문도 있을것 같다고 하심)
- HBASE로 받는 이유
  - 처음 hdfs -> 사용자 많아짐 / 안정성 떨어짐 -> hbase로 전환


## TRACK1 Pinpoint는 어떻게 observability를 강화했는가
구태진님

[발표자료](https://deview.kr/data/deview/2019/presentation/[212]핀포인트는+어떻게+observability를+강화했는가.pdf)

Pinpoint 릴리즈 노트 / 구현방법을 빠르게 훑는 강연 
-> 발표 자료를 보는게 좋음

Monitoring : 
Opservability : 서비스에 대한 이해 / 장애대응에 대한 insight제공
- 핀포인트 고도화에 따
핀포인트
- 핀포인트 코어가 궁금하면 데뷰 2015를 보면 됨
문제
- 현재 각 에이전트가 처리하고 있는 요청은 얼마나 되는지?
- 예상하지 못한 문제로 끝나지 않는 요청이 있다면 어떻게 확인하는지?
구조
- 핀포인트 패킷 
 - 데이터 전송의 최소 단위
- 내가 이걸 왜 듣고있는지 잘 모르겠다
- 잘못 들어옴 

- pinpoint의 구현
- 라이브 릴리즈 노트
- 
- 데이터 패턴 분석 -> 거의 비슷 / 반복이 많음 -> delta / delta of delta / repeat 등 활용
- 핀포인트 고도화 설명


## TRACK2 Fail Fast, Learn Faster SRE (실패에서 배워나가는 SRE)
김재헌님 / 유호균님

[발표자료](https://deview.kr/data/deview/2019/presentation/[223]Fail%20Fast,%20Learn%20Faster%20SRE%20(%EC%8B%A4%ED%8C%A8%EC%97%90%EC%84%9C%20%EB%B0%B0%EC%9B%8C%EB%82%98%EA%B0%80%EB%8A%94%20SRE)_comp.pdf)


- SRE란?
  - SRE: Site Reliability Engineering
  - 사이트 신뢰성 엔지니어링
  - 네이버 검색 SRE팀의 정의
    - 신뢰성: 네이버 검색 서비스가 정상적으로 제공되는 것
    - 방법론: Availability, Capacity Planning, Monitoring, Contingency Plan
    - 문화 : Post-Mortem, Blameless, Fact-Based

- 왜 SRE를 도입해야 하는가  
  - 온 국민이 사용하는 공공재
  - 서비스가 느는 만큼 엔지니어 수는 늘지 않음
  - DR : 지진과 같은 재난상황에는 기존의 대응방안이 동작하지 않는다.
  - 경주 지진 당시 통합검색 장애
    - 기존의 DevOps 방법론으로는 규모도, 재난도 감당할 수 없는 현실
  - 16년 말 도입 결정

- 우리들의 SRE
  - 많은 도구 -> 현실에 닥친 문제 해결하기
  - 문제점
    - 1. 비상상황 대응 체계의 부재
      - 정의가 없었음 
      - 뭘 해야하는 지 대응체계가 없음
    - 2. 장애 발생 Detection의 어려움
      - 장애 탐지 기준이 필요 (쉬운 기준)
      - 가용량 지표 개발
    - 3. Post-Mortem 문화 부재
      - 사후처리 Report 누락 / 비슷한 유형 장애 재발시 대응이 잘 안되었음 
    - 4. 전체 시스템 현황 확인 불가
      - 다양한 서비스의 연계 현황 확인 필요

- Metric + Meta Data = Insight
  - 기존에도 잘 되던 것들
    - 수십가지 지표 (실셈 지표 / 서비스 지표)
    - 분당 수십만 ~수백만
    - 어떻게 하면 꼭 필요한 Insight를 제공하는가?
    - 1. 서비스 단위 정보 자동 Aggregation
      - SSUID 발급
      - 잘 정의하면 서비스 단위의 Alerting 가능
    - 2. 담당자 정보의 정확도
      - DRI (서비스 담당자 정보)
      - 경보에 특화된 Meta Data API를 만들고, 강제성 부여
      - 당당자를 지정하지 않으면 알림을 받을 수 없음
      - 정 / 부
  - 다양한 메타데이터 활용하기
    - Problem: 수만대의 서버 중 테스트 서버가 섞여 있으면 합산 지표에 왜곡이 발생
    - Solution: 장비별로 개발 스테이지 테스트 프로덕션 구분
    - 이걸? 당연한거 아닌가...

- Fail Fast, Learn Faster
  - 결국 SRE는 근본적으로 장애와 싸우는 일
  - [장애 복구, 원인 분석, 재발 방지 대책]에 소요되는 시간은 줄이기 어려움
  - 그렇지만 [이상징후 탐지, 의사결정, 영향도 파악] 시간은 줄일 수 있음
  - 경보를 줄이기 위한 개발이 아니라 원칙이 필요
    - 원칙 1 간단함
      - 오캄의 면도날
      - 가능하면 간단한 추론을 하라 / 조건이 같다면 간단한 것이 더 강력하다
      - 임계상황 판단 공식
        - 부하증가배수 : 한 서버가 죽으면 나머지 서버들은 몇 배의 부하를 받는가?
        - 최대가용배수 : 한 서버가 지금의 몇 배까지 받을 수 있는가? 
    - 원칙 2 하인리히 법칙 (1:29:300)
      - 대형사고가 일어나기 전에는 수십차례의 경미한 사고, 수백번의 징후가 반드시 일어난다
      - 적정한 수준의 경보 빈도
      - 위 비율에 맞추어 경보 숫자 튜닝을 진행
    - 평일이면서 공휴일이었던 날 
      - 평소의 양상과 다른 패턴의 트래픽 -> 알림 발생
      - 기존 규칙이 아니라 따로 통계 및 학습을 통해 적용
  - 적용해서 거짓 경보들을 많이 감소시킴
  - 고급
    - 트래픽 패턴 분석
      - Traffic Migration: 트래픽(Traffic)이 다른 곳으로 이전(Migration) 되었는지 판단
    - 자동 비상 대응
      - 포항 지진 이후 자동 비상 대응 시스템 개발 시작
      -  if (대한민국 4.0 이상 지진 발생 OR 통합검색 트래픽의 급격한 증가) {
           Cache Expire Time 연장 -> 서버 부하 방지
         }
      - 이후 2차 포항 지진시 자동 방어장치 가동
  - 요약
    - 줄일 수 있는 시간을 최대한 줄인다
    - 단순하고 효과적인 방법으로 빠르게 실패하고 빠르게 배워서 개선한다.
  - SRE == DR
    - 본질적인 SRE에 집중하다 보면 결국 재난 엔지니어링으로 귀결된다


## TRACK2 React, Angular, Vue를 한 번에 지원하기 위한 설계 (Cross Framework Component)
최연규님

[발표자료](https://deview.kr/data/deview/2019/presentation/[224]+React,+Angular,+Vue%EB%A5%BC+%ED%95%9C+%EB%B2%88%EC%97%90+%EC%A7%80%EC%9B%90%ED%95%98%EA%B8%B0+%EC%9C%84%ED%95%9C+%EC%84%A4%EA%B3%84-191025.pdf)

- Cross Framework Component
  - 하나의 모듈 -> 여러 프레임웍 지원
  - 크로스 플랫폼 : 1 개발 -> 여러 플랫폼에서 사용
    - 자마린 플러터 네이티브 스크립트
  - 1 js -> 모든 framework
- 바닐라 컴포넌트를 프레임웍에 적용한 사례 /문제점
  - 단순하게 래핑
    - 내부 변수에 접근시 데이터 동기화가 안됨
  - 별도로 만들기
    - 비슷한 형태의 코드가 여러벌 만들어짐
  - 고민과 해결
    - 결국엔 DOM 조작의 문제다
- 프레임워크 DOM 렌더링 원리(key & DOM Diff)
  - 다른 문법 다만 key가 동일
- Cross Framework Component(CFC) 원리
  - DOM 조작 외부화
    - 데이터만 조작 / 프레임워크에게 돔 조작을 위임
  - 추가 / 삭제 메소드 호출 원리
    - ListDiffer : 데이터의 추가 / 삭제 / 유지 여부를 알려주는 라이브러리를 만듦
- Cross Framework Component(CFC) 적용방법
  - 기본 : 추가/삭제 메소드 제공
  - DOM 조작 외부화 : 렌더링 외부화를 옵션을 통해 제공
  - List의 Diff를 알 수 있는 메소드 제공
    - removed -> maintained -> added 순으로 조사
- 누구나 쉽게 만들 수 있는 InfiniteScroll에 CFC 적용해보기 / egjs에서 CFC를 적용한 사례
  - https://github.com/NAVER-FEPlatform/deview2019-demo


## TRACK2 2019년 FE 프레임워크를 배우는 기분(FE 인싸들이라면 알고 있어야 하는 프레임워크 기술들)

[발표자료](https://deview.kr/data/deview/2019/presentation/[225]+2019%EB%85%84+FE+%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EB%A5%BC+%EB%B0%B0%EC%9A%B0%EB%8A%94+%EA%B8%B0%EB%B6%84(FE+%EC%9D%B8%EC%8B%B8%EB%93%A4%EC%9D%B4%EB%9D%BC%EB%A9%B4+%EC%95%8C%EA%B3%A0+%EC%9E%88%EC%96%B4%EC%95%BC+%ED%95%98%EB%8A%94+%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC+%EA%B8%B0%EC%88%A0%EB%93%A4)-191028.pdf)


## TRACK3 Armeria: 어디서나 잘 어울리는 마이크로서비스 프레임워크
이희승님

[발표자료](https://deview.kr/data/deview/2019/presentation/[236]2019.10.%20Armeria%20-%20A%20Microservice%20Framework%20Well-suited%20Everywhere.pdf)