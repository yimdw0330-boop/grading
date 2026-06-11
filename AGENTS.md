# AI Agent Guide for grading

## 프로젝트 개요
- 목적: 수학 학원 학생들의 숙제/시험 채점용 단일 페이지 웹앱.
- 배포: GitHub Pages에 배포되는 정적 앱.
- 주요 파일: `index.html`에 모든 HTML/CSS/JavaScript가 포함됨.
- 데이터 흐름: 브라우저 JS → GAS 웹앱 엔드포인트 → Google Sheets에 채점 데이터 저장.

## 실제 설정값
- `GAS_URL`: `https://script.google.com/macros/s/AKfycbyMnbbiat1wf77YHc3iYqzfT-MuEb_-BR7VYismnNDSBFn1mY4deohvRMbl58AN6t-t/exec`
- `WORKERS_URL`: `https://restless-queen-374b.yimdw0330.workers.dev`
- Google 스프레드시트 ID: 이 클라이언트 코드(`index.html`)에는 직접 포함되어 있지 않음. 시트 ID와 세부 저장 로직은 GAS 앱 쪽에서 관리됨.

## 백엔드(GAS) 작업 방식
- GAS(`Code.gs`)는 Google Apps Script 환경에서 별도 관리됨.
- Claude Code는 GAS 파일에 직접 접근 불가.
- 백엔드 변경이 필요하면 GAS 코드를 새로 작성해서 보여주고, 사용자가 Apps Script 편집기에 수동으로 붙여넣음.
- 기존 GAS action 목록:
  - `getQuizList`, `getAnswers`, `saveResult`
  - `teacherLogin`, `getPendingTeacher`, `saveTeacherGrade`
  - `getStudentResults`, `getAllResults`, `getQuizAnswers`, `updateAnswer`
  - `saveQuiz` (PDF 등록용)

## 핵심 방향 — 주관식의 객관식 변환
- 이 앱은 주관식 문제를 객관식 형태로 자동 변환하는 채점 방식을 지향함.
- 정답 하나를 입력하면, 수치적으로 유사한 오답을 자동 생성해 보기 5개를 만듦.
- 오답 생성 규칙:
  - 분수면 분모/분자를 살짝 바꿈
  - 약분 전 형태 사용
  - 부호 반전 또는 자리수 실수 형태
  - 실제 오류 유형을 반영한 오답 생성
- 정답 보기 위치(1~5번)는 문제마다 무작위로 섞음. 정답 자리 패턴 방지 목적.
- 6번 보기 고정: 항상 `모름`.
  - `모름`은 단순 오답이 아니라 미응답/미학습 신호로 처리.
  - 학생이 찍지 않고 모르면 모른다고 표시하도록 유도.

## 현재 구현 상태
- 학생 모드: 시험지 선택, 이름 입력, 답안 입력, 채점, 결과 요약, 재채점 기능.
- 선생님 모드: 비밀번호 로그인, 채점 대기 확인, 채점 결과 확인, 정답 수정, PDF/이미지 업로드로 정답 등록.
- PDF/이미지 업로드는 `WORKERS_URL`에 파일을 보내고 AI가 답안을 추출한 뒤 GAS로 저장함.

## 현재 진행 중인 작업
- 학생 명단 선택 기능 추가 (드롭다운 + 새 이름 추가).
- 신규 GAS action: `getStudentNames`, `addStudentName`.
- Sheets에 `명단` 시트 사용 (A열: 이름).

## 다음 작업(로드맵)
- 답안지 PDF에서 정답을 일괄 추출해 Google Sheets에 등록하기.
  - 대상: M3.1 기본100, M3.1 표준119, M3.2 기본77, M3.2 표준114, 3가년 단문형72
- 앱에 객관식 자동 변환 기능 구현하기.

## 향후 데이터 구조 변경 (객관식 변환 시)
- 변환된 문제는 `보기1`~`보기6` 컬럼 + `정답위치`(1~5 중 어디로 섞였는지) 컬럼 필요.
- 6번은 항상 "모름" 고정이므로 별도 컬럼 불필요.
- 채점 로직: 학생 답 == 정답위치 → 정답, 6번 선택 → "모름" 별도 집계.

## 중요 규칙
- 한국어 수학 PDF의 수식은 텍스트가 아니라 이미지임.
- 정답 추출 시 `pdftotext` 같은 텍스트 추출이 아니라 화면을 직접 눈으로 읽어서 처리해야 함.
- 소통은 한국어로 간결하게, 바로 쓸 수 있는 결과물 위주로.

## AI 에이전트용 지침
- 변경은 `index.html` 안에서 self-contained로 처리.
- UI나 로직 추가 시 기존 CSS 변수와 클래스 재사용.
- 백엔드 계약이 필요한 경우, GAS 엔드포인트에서 사용하는 `action` 값과 요청 본문을 먼저 확인.
- Google Sheets ID는 코드에 없으므로, 그 정보가 필요하면 GAS 앱 서버 쪽 코드를 확인해야 함.