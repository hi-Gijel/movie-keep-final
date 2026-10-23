# SQL 실행 결과
실행 순서: 완성한 00-setup → 01-queries → 00-setup → 02-crud → 00-setup → 03-constraints.
각 항목에 실제 실행한 SQL과 실제 결과(텍스트 또는 캡처)를 첨부하세요.
예상 결과를 그대로 복사한 것만으로 실행을 확인할 수 없습니다.

## 준비: 외래 키 1 / 영화 6건 / 찜 8건

실행 순서: 완성한 00-setup 실행 (최초 1회)

- 실행 SQL:
PRAGMA foreign_keys;
SELECT * FROM movies ORDER BY id ASC;
SELECT * FROM favorites ORDER BY id ASC;

╭──────────────╮
│ foreign_keys │
╞══════════════╡
│            1 │
╰──────────────╯
╭─────┬────────────────────╮
│ id  │       title        │
╞═════╪════════════════════╡
│ 101 │ 인셉션             │
│ 102 │ 인터스텔라         │
│ 103 │ 다크 나이트        │
│ 104 │ 테넷               │
│ 105 │ 어벤져스           │
│ 106 │ 어벤져스: 엔드게임 │
╰─────┴────────────────────╯
╭────┬────────┬─────────┬─────────────────────╮
│ id │ userId │ movieId │        memo         │
╞════╪════════╪═════════╪═════════════════════╡
│  1 │ user-a │     101 │ 주말에 볼 영화      │
│  2 │ user-b │     101 │ 친구가 추천한 영화  │
│  3 │ user-a │     102 │                     │
│  4 │ user-c │     103 │ 다시 보고 싶은 영화 │
│  5 │ user-a │     104 │ 가족과 볼 영화      │
│  6 │ user-b │     105 │                     │
│  7 │ user-c │     101 │ OST가 좋은 영화     │
│  8 │ user-a │     106 │ 친구와 볼 영화      │
╰────┴────────┴─────────┴─────────────────────╯


## 조건 조회 05·06·07·08

실행 순서: 00-setup → 01-queries


- 실행 SQL (05):
SELECT userId, memo FROM favorites WHERE movieId = 101;

╭────────┬────────────────────╮
│ userId │        memo        │
╞════════╪════════════════════╡
│ user-a │ 주말에 볼 영화     │
│ user-b │ 친구가 추천한 영화 │
│ user-c │ OST가 좋은 영화    │
╰────────┴────────────────────╯


- 실행 SQL (06):
SELECT memo FROM favorites WHERE userId = 'user-a' AND movieId = 104;

╭────────────────╮
│      memo      │
╞════════════════╡
│ 가족과 볼 영화 │
╰────────────────╯

- 실행 SQL (07):
SELECT movieId, memo FROM favorites WHERE userId = 'user-c' ORDER BY movieId DESC LIMIT 1;

╭─────────┬─────────────────────╮
│ movieId │        memo         │
╞═════════╪═════════════════════╡
│     103 │ 다시 보고 싶은 영화 │
╰─────────┴─────────────────────╯


- 실행 SQL (08):
SELECT movieId, memo FROM favorites WHERE userId = 'user-c' ORDER BY movieId DESC;

╭─────────┬─────────────────────╮
│ movieId │        memo         │
╞═════════╪═════════════════════╡
│     103 │ 다시 보고 싶은 영화 │
│     101 │ OST가 좋은 영화     │
╰─────────┴─────────────────────╯


## 빈 메모 JOIN·전체 집계·사용자별 집계 14·15·16

실행 순서: 00-setup → 01-queries


- 실행 SQL (14):
SELECT f.userId, m.title, f.memo FROM favorites f JOIN movies m ON f.movieId = m.id WHERE f.memo = '' ORDER BY f.id ASC;

╭────────┬────────────┬──────╮
│ userId │   title    │ memo │
╞════════╪════════════╪══════╡
│ user-a │ 인터스텔라 │      │
│ user-b │ 어벤져스   │      │
╰────────┴────────────┴──────╯


- 실행 SQL (15):
SELECT COUNT(*) AS emptyMemoCount FROM favorites WHERE memo = '';

╭────────────────╮
│ emptyMemoCount │
╞════════════════╡
│              2 │
╰────────────────╯


- 실행 SQL (16):
SELECT userId, COUNT(*) AS emptyMemoCount FROM favorites WHERE memo = '' GROUP BY userId;

╭────────┬────────────────╮
│ userId │ emptyMemoCount │
╞════════╪════════════════╡
│ user-a │              1 │
│ user-b │              1 │
╰────────┴────────────────╯

## CRUD 09·10·11
추가: 9건 / 수정: id 30의 memo만 변경 / 삭제: 초기 8건 복원

실행 SQL (09):
INSERT INTO favorites (id, userId, movieId) VALUES (30, 'user-b', 106);
- 실행 결과: 오류 없음 (INSERT는 결과 표가 나오지 않음)
- 확인 SELECT: SELECT COUNT(*) AS favoriteCount FROM favorites; → 9

실행 SQL (10):
UPDATE favorites SET memo = '주말에 다시 보기' WHERE id = 30;
- 실행 결과: 오류 없음
- 확인 SELECT: SELECT * FROM favorites WHERE id = 30; → memo가 '주말에 다시 보기'

실행 SQL (11):
DELETE FROM favorites WHERE id = 30;
- 실행 결과: 오류 없음
- 확인 SELECT: SELECT COUNT(*) AS favoriteCount FROM favorites; → 8


## 제약조건 A·B·C·D
각 SQL, 성공/실패, 실제 오류 메시지, 이유를 기록하세요.
D 직후 찜 9건과 영화 106 유지, 정리 후 초기 8건을 확인하세요.

실행 순서: 00-setup 재실행 완료 → 03-constraints


A. 실행 SQL: INSERT INTO favorites (id, userId, movieId) VALUES (40, 'user-a', 101);
   성공/실패: 실패
   실제 오류 메시지: Error near line 1: UNIQUE constraint failed: favorites.userId, favorites.movieId
   이유: 같은 사용자가 같은 영화를 중복 찜함 (UNIQUE)

B. 실행 SQL: INSERT INTO favorites (id, userId, movieId) VALUES (41, 'user-b', 106);
   성공/실패: 성공
   실제 오류 메시지: 없음
   이유: 같은 영화를 다른 사용자가 찜하는 것은 허용됨

C. 실행 SQL: INSERT INTO favorites (id, userId, movieId) VALUES (42, 'user-b', 999);
   성공/실패: 실패
   실제 오류 메시지: Error near line 1: FOREIGN KEY constraint failed
   이유: 존재하지 않는 영화 참조 (FOREIGN KEY)

D. 실행 SQL: DELETE FROM movies WHERE id = 106;
   성공/실패: 실패
   실제 오류 메시지: Error near line 1: FOREIGN KEY constraint failed
   이유: 찜이 참조 중인 영화 삭제 (FOREIGN KEY).영화 삭제가 차단되어 106이 그대로 유지됨

D 직후 확인 (찜 9건, 영화 106 유지):
SELECT COUNT(*) AS favoriteCount FROM favorites; → 9
SELECT * FROM movies WHERE id = 106; → 106 / 어벤져스: 엔드게임 1행

정리 후 확인 (초기 8건):
DELETE FROM favorites WHERE id = 41;
SELECT COUNT(*) AS favoriteCount FROM favorites; → 8

