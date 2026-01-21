# Security News Sender

AI 기반 보안 뉴스 필터링 및 Slack 발송 봇

## 주요 기능

- RSS 피드에서 보안 뉴스 수집
- AI(Claude/GPT)를 활용한 지능형 필터링
- 관련 기사만 자동으로 Slack에 발송
- 중복 기사 자동 제거
- 프롬프트 변경 시 자동 재검토 기능
- 실행 로그 자동 저장 및 관리

## Slack 알림 예시

다음과 같은 형태로 Slack에 보안 뉴스 알림이 발송됩니다:

![Slack 알림 예시](./images/slack-notification-example.png)

**알림 형식 (아이콘 최소화):**
- 🔴 위험도 표시만 사용 (높음/중간/낮음)
- 나머지는 텍스트 라벨로 표시 (가독성 향상)
- 대상 시스템/기업 정보
- AI 판단 점수 및 결정
- 기사 발행일
- 기사 요약 정보

## 설치

```bash
# 가상환경 생성 및 활성화
python -m venv venv
source venv/bin/activate  # macOS/Linux
# 또는
venv\Scripts\activate  # Windows

# 의존성 설치
pip install -r requirements.txt
```

## 환경변수 설정

`.env` 파일을 생성하고 다음 환경변수를 설정하세요:

```bash
# 필수
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# AI 사용 시 (Claude)
USE_AI_JUDGMENT=true
AI_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-3-5-haiku-20241022

# 또는 AI 사용 시 (OpenAI)
USE_AI_JUDGMENT=true
AI_PROVIDER=openai
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
```

## 사용법

```bash
# 스크립트 실행
python aitest.py
```

## 필터링 기준

AI가 다음 **세 가지 경우에만** 기사를 발송합니다:

### ✅ 발송 대상 (SCRAPE: 80-100점)

1. **국내 기업 해킹 및 정보유출사고**
   - 한국 기업의 실제 해킹/정보유출/랜섬웨어 공격 사고
   - 국내 기업이 해외 조직(북한, 중국, 러시아 등)으로부터 공격받은 사고
   - ⚠️ 제외: 해외 뉴스, 기관 사고, 직업군 단위 유출, 분석/리뷰, 수사 진행 상황, 보안 인식/교육 기사

2. **개인정보 관련 핵심 법령 개정 및 강화**
   - 개인정보보호법, 정보통신망법, 신용정보법, 위치정보법 개정/강화
   - 개인정보보호위원회(개보위) 가이드라인/지침 발표/개정
   - 과징금 기준 변경, 처벌 강화, 의무 사항 추가 등 법령 강화
   - ISMS-P 인증 기준 변경/강화
   - ⚠️ 제외: 법령 개정이 아닌 단순 설명/행사/의견 기사, 해외 법령 개정

3. **보안 담당자가 궁금해할 취약점**
   - **우리가 사용하는 서비스/제품**: Adobe Reader, FortiGate Firewall, Windows, Office 365
   - **널리 사용되는 서비스/제품**: 아파치(Apache) 관련 오픈소스, 널리 사용되는 오픈소스 라이브러리/프레임워크, 널리 사용되는 네트워크 장비
   - **실무에서 자주 사용하는 SaaS 서비스**: LinkedIn, Google Gemini, Google Workspace, Slack, Zoom 등 (실제 취약점만)
   - Windows 프로토콜/서비스 취약점 포함 (NTLM, SMB, RDP 등)
   - ⚠️ Office 365만 허용 (Office 2016/2019/2021 등 설치형 제외)
   - ⚠️ 피싱/악성코드 유포 기사는 제외 (보안 인식/교육 범주)

### ❌ 제외 대상 (SKIP: 0-49점)

- 해외 뉴스 (단, 우리 시스템 취약점은 해외 뉴스여도 포함)
- 일반 뉴스/홍보/분석 기사
- 보안 인식/교육 기사
- 우리 시스템과 무관한 취약점

### ⚠️ 의심/가능성 표현 (WATCHLIST: 50-79점)

- "의심", "가능성", "추정"만 있고 사실 확인이 안 된 경우

---

📖 **상세 판단 기준**: [AI_JUDGMENT_CRITERIA_SUMMARY.md](./AI_JUDGMENT_CRITERIA_SUMMARY.md) 참고

## GitHub Actions

이 프로젝트는 GitHub Actions를 통해 자동으로 실행됩니다:

- **실행 주기**: 매 30분마다 자동 실행
- **수동 실행**: GitHub Actions 탭에서 `workflow_dispatch`로 수동 실행 가능
- **상태 파일**: `state.aitest.json`이 자동으로 커밋되어 실행 상태 추적
- **발송 로그**: `debug_sent_entries.json`에 발송된 기사 정보 저장
- **실행 로그**: `logs/execution_YYYYMMDD_HHMMSS.log`에 실행 로그 저장 (타임스탬프 포함)

## 프롬프트 변경 시 재검토 기능

프롬프트 파일(`ai_prompt_simple.txt`)이 변경되면 자동으로 감지하여:
- 이미 제외된 기사들을 새로운 기준으로 재검토
- 재검토 결과만 확인 (재발송하지 않음)
- 새로운 기준에서 제외가 맞는지 확인 가능

**재발송 방법**: `state.aitest.json`에서 해당 기사 링크를 `seen_links`에서 제거하면 다음 실행 시 발송됩니다.

## 파일 구조

- `aitest.py`: 메인 스크립트
- `ai_prompt_simple.txt`: AI 판단 기준 프롬프트
- `state.aitest.json`: 처리된 기사 추적 상태 파일
- `debug_sent_entries.json`: 발송된 기사 로그 (GitHub Actions 실행 시 자동 생성)
- `logs/execution_*.log`: 실행 로그 파일 (타임스탬프 포함)
- `.github/workflows/security-news-bot.yml`: GitHub Actions 워크플로우 정의
- `requirements.txt`: Python 패키지 의존성


## 라이선스

Internal use only
