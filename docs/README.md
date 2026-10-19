# Atlas 문서

> Atlas 제품군의 공개 사용자 문서예요. 지금은 **프리뷰(0.x)** 단계이고, **2026년 12월 정식 오픈**과 함께 소스도 공개할 예정이에요.
> 설치 파일은 [Releases](https://github.com/oranewbie/atlas-dist/releases) 에 있어요.

## Atlas 는 무엇인가요

**사람과 AI 가 한 프로젝트 안에서 같은 방법으로 함께 일하게 해 주는 도구 모음**이에요.
일하는 방법을 단계·절차서·도구로 담아 두면, AI 가 그 방법대로 일하고, 사람은 방향을 정하고 결과를 확인해요.
방법은 쓸수록 다듬어지고, 다음 프로젝트로 그대로 가져갈 수 있어요.

```
                ┌──────────────────────────────┐
                │        Atlas Studio          │  ← 사람이 쓰는 데스크톱 앱
                │  (작업대 · 편집기 · AI 챗)      │
                └───────┬───────────────┬──────┘
          AI 를 돌릴 때   │               │   데이터 파이프라인을 돌릴 때
                ┌───────▼──────┐ ┌──────▼───────┐
                │ atlas-agent  │ │ atlas-engine │
                │  AI 에이전트   │ │ 그래프 실행기  │
                └──────────────┘ └──────────────┘
                ┌──────────────────────────────┐
                │         atlas-react          │  ← 업무 웹 화면을 만드는 UI 프레임워크
                └──────────────────────────────┘     (AI 가 화면을 직접 다룰 수 있게)
```

## 제품

| 제품 | 한 줄 소개 | 누구에게 | 최신 버전 | 문서 |
|---|---|---|---|---|
| **Atlas Studio** | 일하는 방법을 담아 AI 와 함께 수행하는 데스크톱 작업대 | 누구나 — 문서·보고서·전환 프로젝트·테스트·개발 | v0.1.29 (Windows) | [시작하기](studio/README.md) |
| **atlas-agent** | 파일·명령·웹·데이터 도구를 쓰는 AI 에이전트 서버. 사내·로컬 모델도 지원 | Studio 사용자, 자기 앱에 AI 를 붙이려는 개발자 | v0.1.4 | [시작하기](agent/README.md) |
| **atlas-engine** | 노드와 선으로 이은 그래프를 실행하는 가벼운 엔진. AI 가 직접 다루는 MCP 제공 | 데이터 파이프라인·자동화를 만드는 분 | v0.1.3 | [시작하기](engine/README.md) |
| **atlas-react** | 조회·편집·저장형 업무 화면을 빠르게 만드는 React 프레임워크 | 업무 시스템 프론트엔드 개발자 | v1.1.5 | [시작하기](react/README.md) |

## 어디서부터 읽을까

| 나는… | 이 순서로 |
|---|---|
| **처음이에요. 뭘 할 수 있는지 궁금해요** | [Studio 소개](studio/README.md) → [활용 사례](studio/use-cases.md) |
| **바로 써 보고 싶어요** | [쉽게 따라하기 (30분)](studio/tutorial.md) |
| **Studio 를 쓰는 중이에요** | [사용자 매뉴얼](studio/manual.md) · [단축키](studio/shortcuts.md) · [문제 해결](studio/troubleshooting.md) |
| **사내 AI 모델을 쓰고 싶어요** | [atlas-agent 설정 가이드](agent/configuration.md) |
| **데이터 파이프라인을 만들고 싶어요** | [atlas-engine 소개](engine/README.md) → [그래프 만들기](engine/graph-guide.md) |
| **업무 화면을 개발해요** | [atlas-react 소개](react/README.md) → [핵심 개념](react/concepts.md) → [컴포넌트](react/components.md) |

## 전체 문서 목록

**Atlas Studio**
- [소개](studio/README.md) · [활용 사례](studio/use-cases.md) · [쉽게 따라하기](studio/tutorial.md) · [용어집](studio/glossary.md)
- [사용자 매뉴얼](studio/manual.md) · [단축키](studio/shortcuts.md) · [문제 해결](studio/troubleshooting.md)

**atlas-agent**
- [소개·설치](agent/README.md) · [설정 가이드](agent/configuration.md) · [기능·API 레퍼런스](agent/reference.md)

**atlas-engine**
- [소개·설치](engine/README.md) · [그래프 만들기 가이드](engine/graph-guide.md) · [API·설정 레퍼런스](engine/reference.md)

**atlas-react**
- [소개·시작하기](react/README.md) · [핵심 개념](react/concepts.md) · [컴포넌트 안내](react/components.md)

**공통**
- [버전과 변경 이력](changelog.md)

## 알아 두세요

- 모든 제품이 **프리뷰** 단계라 화면과 기능이 자주 바뀌어요. 문서는 각 제품의 위 표에 적힌 버전을 기준으로 해요.
- Studio 앱 안의 **도움말 › 도움말 문서…** 에는 그 앱 버전에 맞는 문서가 들어 있어요.
- 소스 코드는 2026년 12월 정식 오픈 때 공개할 예정이에요. 그 전까지 이 저장소에는 설치 파일과 문서만 있어요.
