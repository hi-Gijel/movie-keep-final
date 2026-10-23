 같은 영화 찜 주제를 바탕으로 세 가지 실습을 각각 독립적으로 완성한 결과물입니다. 세 실습은 서로 연결하지 않았습니다. Firebase-SQLite 동기화, Express DB 연결, 프런트엔드와 Express API 연결은 구현하지 않았습니다.

폴더 구조
firebase-app/: 실습 A, 로그인·찜·메모 기능을 구현한 Next.js 앱
sql/: 실습 B, 00-setup.sql, 01-queries.sql, 02-crud.sql, 03-constraints.sql
express-api/: 실습 C, 찜 목록·개수 API 서버
results/: SQL 결과 작성양식, 본인 README, 캡처 이미지 4장

실행 방법
실습 A (Firebase)

Copycd firebase-app
npm ci
cp .env.example .env.local
npm run dev
.env.local에는 본인 Firebase 프로젝트 설정값과 TMDB 토큰을 입력했습니다. 환경 변수를 수정한 뒤에는 개발 서버를 재시작했습니다. 브라우저에서 http://localhost:3000/movies를 열어 확인했습니다.

실습 B (SQL)

SQLite Fiddle(https://sqlite.org/fiddle/index.html)에서 실행했습니다. 00-setup.sql로 외래 키 1, 영화 6건, 찜 8건을 준비한 뒤 나머지 파일을 순서대로 실행했습니다. 각 단계 전에는 00-setup.sql로 초기화했습니다.

실습 C (Express)

Copycd express-api
npm ci
npm run dev
http://localhost:4000/에서 기본 메시지를, /api/favorites와 /api/favorite-count에서 응답을 확인했습니다.

설명
1. 새로고침 후 Firebase 데이터를 Store에 다시 반영하는 이유

Store는 브라우저 메모리에만 있어서 새로고침하면 초기화됩니다. 그래서 로그인 상태가 확인되면 Firestore에서 찜 목록을 다시 조회해 Store에 채워야 화면이 이전 상태로 복원됩니다. 실제 데이터의 원본은 Firestore이고, Store는 화면 표시를 위한 사본이라고 이해했습니다.

2. DB 저장 성공 후 Store를 갱신하는 이유

Store를 먼저 바꾸면 저장이 실패했을 때 화면에는 저장된 것처럼 보이지만 DB에는 반영되지 않는 불일치가 생깁니다. 그래서 updateDoc()이 성공한 뒤에만 Store를 갱신하도록 순서를 정했습니다.

3. 빈 메모 저장과 찜 삭제의 차이

빈 메모 저장은 찜 문서는 그대로 두고 memo 필드만 빈 문자열로 바꾸는 작업입니다. 찜 삭제는 문서 자체를 없애는 작업입니다. 메모를 지워도 영화는 찜 목록에 남아 있어야 하므로 두 작업은 대상과 결과가 다릅니다. 실제로 빈 메모를 저장한 뒤에도 찜이 유지되는 것을 확인했습니다.

4. 복합 UNIQUE와 외래 키의 역할

복합 UNIQUE(userId, movieId)는 같은 사용자가 같은 영화를 두 번 찜하지 못하도록 막습니다. 외래 키(movieId가 movies.id를 참조)는 존재하지 않는 영화를 찜하거나, 찜이 참조하는 영화를 삭제하는 것을 막습니다. 전자는 중복을, 후자는 데이터 연결의 무결성을 지킵니다. 제약조건 검증에서 A(중복)와 C(없는 영화)는 실패, B(다른 사용자의 같은 영화)는 성공, D(찜이 있는 영화 삭제)는 실패하는 것을 확인했습니다.

5. 전체 집계와 사용자별 집계의 차이

전체 집계(COUNT(*))는 조건에 맞는 행이 모두 몇 개인지 하나의 값으로 보여 줍니다. 사용자별 집계는 GROUP BY userId로 묶어 사용자마다 따로 개수를 셉니다. 빈 메모 전체 개수는 2건이지만, 사용자별로 나누면 user-a 1건, user-b 1건으로 나뉘어 보였습니다.

6. API 개수를 배열 길이로 계산하는 이유

개수를 숫자 8로 직접 적으면 데이터가 바뀌어도 응답이 그대로 남아 실제 상태와 어긋납니다. favorites.length로 계산하면 배열이 바뀔 때마다 개수도 자동으로 맞춰집니다. 테스트 항목을 추가했을 때 개수가 9로 바뀌는 것으로 이 동작을 확인했습니다.

구현 내용
실습 A: TODO 01(로그인 사용자 Store 반영), TODO 02(찜 목록 조회 후 Store 반영), TODO 03-a/b(updateDoc으로 메모 저장), TODO 04(memo 기본값 ?? 처리)를 구현했습니다. 로그인, 찜 복원, 메모 저장·수정·복원, 빈 메모 저장 후 찜 유지, 로그아웃 시 초기화를 확인했습니다.
실습 B: TODO 12·13(복합 UNIQUE·외래 키), TODO 05~08(조건 조회), 09~11(INSERT·UPDATE·DELETE), 14~16(JOIN·집계), 제약조건 A~D를 실행하고 결과를 기록했습니다.
실습 C: TODO 17(/api/favorites), TODO 18(/api/favorite-count)을 구현했습니다. 응답은 찜 8건과 {"favoriteCount":8}입니다.

선택 과제: 구현하지 않았습니다.

캡처 이미지는 results 폴더에 업로드 되어있습니다.