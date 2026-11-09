# 권한 우회 테스트용 클론 — 사용 전 읽어주세요

이 코드는 원본 ANANTHEO 저장소를 복제한 뒤, **클라이언트 쪽 방어를 의도적으로
제거하거나 노출시켜** 서버(Supabase RLS/RPC/Storage 정책)가 실제로 요청을
막아주는지 확인하기 위한 자가진단용 버전입니다.

## 반드시 지켜야 할 것
- **본인 소유 Supabase 프로젝트**에, **테스트 전용 계정**으로만 로그인해서 쓰세요.
- 로그인 화면은 원본과 똑같이 실제 서버로 요청을 보냅니다. 다른 사람에게 이
  클론 링크를 보내서 로그인하게 하면 그건 피싱이 됩니다 — 절대 그렇게 쓰지 마세요.
- `/pentest` 페이지의 각 버튼은 "원래 막혀야 하는 요청"을 실제로 보냅니다.
  실행하면 테스트 계정의 실제 데이터(프로필 role, 포인트, 글 등)가 바뀔 수
  있습니다. 운영 중인 진짜 계정으로는 절대 실행하지 마세요.

## 원본 대비 바뀐 부분

| 파일 | 변경 | 목적 |
|---|---|---|
| `app/admin/page.tsx` | `role === 'admin'` 화면 가드 제거 | `/admin`이 데이터까지 보여주는지, RLS가 막는지 확인 |
| `components/layout/app-shell.tsx` | `is_suspended` 강제 로그아웃 제거, 네비게이션에 VULN-TEST 링크 추가 | 정지 계정이 API를 계속 쓸 수 있는지 확인 |
| `app/study/page.tsx` | 분/날짜 유효성 검사 제거, 제출 시 `status`를 직접 `approved`/`rejected`로 선택 가능 | 서버 쪽 CHECK 제약·상태 강제가 있는지 확인 |
| `app/community/page.tsx` | `is_hidden` 글/댓글 숨김 필터 제거, 글 작성 시 임의 `user_id`(사칭) 입력란 추가 | RLS가 숨김·소유자 검증을 하는지 확인 |
| `app/pentest/page.tsx` (신규) | 9가지 권한 우회 시나리오를 자동 실행하는 진단 페이지 | 한 번에 전체 점검 |

## `/pentest` 테스트 목록
1. 내 `profiles.role`을 `admin`으로 자가 승격
2. `participant_point_balances` / `participant_warning_balances` 뷰 전체 조회(남의 행 노출 여부)
3. `admin_adjust_points`, `admin_set_participant_suspension`, `admin_set_participant_hidden` RPC를 일반 계정으로 호출
4. `study_entries`를 `status=approved`로 바로 제출(승인 절차 우회)
5. 본인 pending 기록을 본인이 직접 승인
6. `community_posts`에 임의의 `user_id`(타인 사칭)로 글 작성
7. `warning_events`에 직접 insert해서 본인 경고 차감
8. Storage 버킷 루트를 `list()`해서 남의 유저 폴더가 보이는지 확인
9. `is_published=false`인 비공개 공지 조회

**빨간 "우회됨"이 하나라도 뜨면** 그 줄의 설명에 적힌 정책(RLS 정책, RPC 내부
관리자 체크, CHECK 제약, Storage 정책)을 추가해야 합니다. 전부 "차단됨"으로
나오면 지금 구조에서는 클라이언트 코드를 아무리 변조해도 넘어갈 수 없다는
뜻입니다.

## 실행 방법
```bash
npm install
npm run dev
```
`.env.local`에 원본과 동일하게 `NEXT_PUBLIC_SUPABASE_ANON_KEY`를 넣으세요.
(테스트 계정으로) 로그인 후 상단 네비게이션의 **⚠️ VULN-TEST**로 이동해
"전체 테스트 실행"을 누르세요.
