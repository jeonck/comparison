<!--
  Request a new IT term/concept comparison here.
  - Edit this file from the GitHub web UI (open this file → pencil icon →
    edit → Commit changes). No local git pull/push needed.
  - Format: one term or "A vs B" topic per line, inside the fenced code block.
  - One line = one post. Multiple lines = one post per line.
  - Saving this file (a commit to main) immediately triggers the pipeline —
    there is no daily schedule and no fallback content.
  - Once a term's post is published, its line is removed automatically —
    this code block goes back to blank, ready for the next request. A term
    that fails generation keeps its line so the next run retries it.
-->
```

오픈소스 AI 에이전트 생태계에서 주목받고 있는 Hermes(Hermes Agent)와 Open Claw(OpenClaw)는 모두 로컬이나 서버에 직접 설치해 사용하는 강력한 도구이지만, 설계 사상과 주력하는 역할에 분명한 차이가 있습니다.   

1.핵심 개념 및 디자인 차이
Open Claw:   

멀티채널 게이트웨이 및 라우팅 중심: 사용자의 로컬 머신이나 서버에 구동되어 텔레그램, 디스코드, 슬랙, 웹 채팅 등 다양한 메신저 채널과 AI 에이전트를 연결하고 관리하는 '게이트웨이(Gateway)' 역할에 초점을 맞춥니다.   

확장성: 다양한 플러그인과 방대한 스킬(ClawHub 등) 생태계를 기반으로 여러 에이전트를 유연하게 조율하는 데 강점이 있습니다.   

Hermes (Nous Research):   

지속형(Persistent) 메모리 및 자기 개선(Self-improving) 중심: Nous Research에서 개발한 오픈소스 에이전트로, 사용자의 워크플로우를 학습하고 시간이 지날수록 능력이 발전하는 '에이전트 인프라(추론 및 메모리 브레인)'에 가깝습니다.   

적응형 학습: 반복되는 복잡한 작업을 자체적으로 스킬화하고 지속형 메모리(SQLite FTS5 등)에 저장하여 사용할수록 사용자 맞춤형으로 고도화됩니다.   

2. 주요 기능 및 특징 비교
구분	Open Claw	Hermes Agent
핵심 목적	다채널 연동 및 멀티 에이전트 라우팅 게이트웨이	장기 실행 자율 작업, 지속형 메모리 및 자기 학습
강점	텔레그램·디스코드·왓섭 등 다양한 메신저 플랫폼과의 뛰어난 연동성 및 생태계	사용자의 작업 패턴을 학습해 시간이 지날수록 정교해지는 적응형 워크플로우
메모리 구조	기본 툴 및 플러그인 의존성이 상대적으로 높음	자체 벡터/Markdown 및 SQLite 기반의 영구 메모리 시스템 구축
생태계 호환성	수많은 서드파티 스킬 및 관리형 플랫폼(OpenClaw Launch 등) 지원	자체 Hermes 모델군 및 OpenAI 호환 API 연동
3. 어떤 것을 선택해야 할까?
  
Open Claw를 선택하기 좋은 경우:   

텔레그램, 디스코드, 왓츠앱 등 여러 메신저 채널을 통해 AI 비서를 실시간으로 연동하고, 다양한 커스텀 스킬과 플러그인을 결합하여 확장성 높은 개인 비서 환경을 구축하고 싶을 때 적합합니다.   

Hermes를 선택하기 좋은 경우:   

매번 반복 설명할 필요 없이 내 작업 스타일과 코딩 패턴을 스스로 기억하고 발전시키는 장기 실행형 에이전트가 필요할 때 이상적입니다. 특히 로컬 환경에서 프라이버시를 지키며 자율적으로 성장하는 인프라를 선호하는 개발자에게 적합합니다.   


```
