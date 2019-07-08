---
title: 카크로치디비 문서
summary: 카크로치디비 사용자 메뉴얼
toc: true
contribute: false
build_for: [standard, managed]
cta: false
---

{% if site.managed %}
관리형 카크로치디비는 카크로치 연구소가 만들고 소유한 완전히 호스팅되고 관리되는 서비스로, 카크로치디비를 손쉽게 배포, 확장 및 관리할 수 있습니다.

{{site.data.alerts.callout_info}}
이 문서는 작업중입니다. 이 곳에서 해소되지 않는 궁금한 점이 있으면 support.cockroachlabs.com](https://support.cockroachlabs.com)으로 연락주십시오.
{{site.data.alerts.end}}

### 언제나 서비스 중

- 클라우드 벤더에 대해 몰라도 됨 Cloud vendor agnostic
- 3개 이상의 데이터 센터에 자동 데이터 복제
- 클라우드 제공업체 간 이동시 마이그레이션 다운타임 없음

### 운영 효율성

- 자동 하드웨어 프로비저닝, 셋업 및 설정
- 자동 롤링 업데이트
- 자동 일일 백업과 시간단위 증분 백업

### 엔터프라이즈급 보안

- 모든 연결에 TLS 1.2 적용
- 전용 클러스터
- SOC-2 인증 (진행 중)

{% else %}


카크로치디비는 글로벌하고 스케일가능한 클라우드서비스를 구축할 수 있는, 재난으로부터 안전한 SQL 데이터베이스입니다.

<div class="container">
  <div class="row display-flex">
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">시작하기</p>
      <div class="landing-column-content">
        <p><a href="install-cockroachdb.html">카크로치디비 설치</a></p>
        <p><a href="start-a-local-cluster.html">로컬 클러스터 실행</a></p>
        <p><a href="learn-cockroachdb-sql.html">카크로치디비 SQL 배우기</a></p>
        <p><a href="build-an-app-with-cockroachdb.html">앱 빌드하기</a></p>
        <p><a href="demo-fault-tolerance-and-recovery.html">기능 살펴보기</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">개발하기</p>
      <div class="landing-column-content">
        <p><a href="install-client-drivers.html">클라이언트 드라이버</a></p>
        <p><a href="connection-parameters.html">커넥션 매개변수</a></p>
        <p><a href="sql-statements.html">SQL 문법</a></p>
        <p><a href="data-types.html">데이터 타입</a></p>
        <p><a href="performance-best-practices-overview.html">SQL 모범 사례</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">배포</p>
      <div class="landing-column-content">
        <p><a href="recommended-production-settings.html">프로덕션 체크리스트</a></p>
        <p><a href="manual-deployment.html">수동 배포</a></p>
        <p><a href="orchestration.html">오케스트레이션</a></p>
        <p><a href="security-overview.html">보안</a></p>
        <p><a href="upgrade-cockroach-version.html">롤링 업그레이드</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">마이그레이션</p>
      <div class="landing-column-content">
        <p><a href="migration-overview.html">개요</a></p>
        <p><a href="migrate-from-postgres.html">Postgres에서 마이그레이션</a></p>
        <p><a href="migrate-from-mysql.html">MySQL에서 마이그레이션</a></p>
        <p><a href="migrate-from-csv.html">CSV에서 마이그레이션</a></p>
        <p><a href="performance-best-practices-overview.html#multi-row-dml-best-practices">삽입 모범 사례</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">문제해결</p>
      <div class="landing-column-content">
        <p><a href="common-errors.html">개요</a></p>
        <p><a href="common-errors.html">일반적인 에러들</a></p>
        <p><a href="cluster-setup-troubleshooting.html">클러스터 셋업</a></p>
        <p><a href="query-behavior-troubleshooting.html">쿼리 동작방식</a></p>
        <p><a href="support-resources.html">지원 리소스</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">래퍼런스</p>
      <div class="landing-column-content">
        <p><a href="sql-feature-support.html">SQL</a></p>
        <p><a href="cockroach-commands.html">CLI</a></p>
        <p><a href="cluster-settings.html">클러스터 설정</a></p>
        <p><a href="admin-ui-overview.html">운영 UI</a></p>
        <p><a href="third-party-database-tools.html">써드파티 툴</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">더 배우기</p>
      <div class="landing-column-content">
        <p><a href="training/">카크로치디비 트레이닝</a></p>
        <p><a href="architecture/overview.html">아키텍처</a></p>
        <p><a href="sql-feature-support.html">SQL 기능 지원</a></p>
        <p><a href="https://www.cockroachlabs.com/guides/">백서</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">FAQs</p>
      <div class="landing-column-content">
        <p><a href="frequently-asked-questions.html">제품 FAQs</a></p>
        <p><a href="sql-faqs.html">SQL FAQs</a></p>
        <p><a href="operational-faqs.html">운영 FAQs</a></p>
        <p><a href="cockroachdb-in-comparison.html">데이터베이스 비교</a></p>
      </div>
    </div>
    <div class="col-xs-12 col-sm-6 col-lg-4">
      <p class="landing-column-title">릴리즈</p>
      <div class="landing-column-content">
        <p><a href="../releases/{{page.release_info.version}}.html">마지막 스테이블 릴리즈</a></p>
        <p><a href="../releases/#production-releases">모든 스테이블 릴리즈</a></p>
        <p><a href="../releases/#testing-releases">모든 테스트 릴리즈</a></p>
        <p><a href="known-limitations.html">알려진 제한사항</a></p>
      </div>
    </div>
  </div>
</div>
{% endif %}
