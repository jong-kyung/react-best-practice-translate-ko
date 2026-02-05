---
name: vercel-composition-patterns
description:
  확장 가능한 React 컴포지션 패턴. 불리언 prop 남발을 줄이기 위해 컴포넌트를
  리팩터링하거나 유연한 컴포넌트 라이브러리를 만들거나 재사용 가능한 API를
  설계할 때 사용합니다. 컴파운드 컴포넌트, render props, 컨텍스트
  프로바이더, 컴포넌트 아키텍처와 관련된 작업에서 트리거됩니다. React 19
  API 변경 사항을 포함합니다.
license: MIT
metadata:
  author: vercel
  version: '1.0.0'
---

# React 컴포지션 패턴

유연하고 유지보수 가능한 React 컴포넌트를 만들기 위한 컴포지션 패턴입니다.
컴파운드 컴포넌트 사용, 상태 끌어올리기, 내부 구성 결합을 통해 불리언 prop
남발을 피합니다. 이 패턴들은 규모가 커질수록 사람과 AI 에이전트 모두가
코드베이스를 더 쉽게 다룰 수 있게 합니다.

## 적용 시점

다음과 같은 경우 이 지침을 참고하세요:

- 불리언 prop이 많은 컴포넌트를 리팩터링할 때
- 재사용 가능한 컴포넌트 라이브러리를 만들 때
- 유연한 컴포넌트 API를 설계할 때
- 컴포넌트 아키텍처를 리뷰할 때
- 컴파운드 컴포넌트 또는 컨텍스트 프로바이더로 작업할 때

## 우선순위별 규칙 카테고리

| 우선순위 | 카테고리               | 영향도 | 접두사          |
| -------- | ---------------------- | ------ | --------------- |
| 1        | 컴포넌트 아키텍처       | HIGH   | `architecture-` |
| 2        | 상태 관리              | MEDIUM | `state-`        |
| 3        | 구현 패턴              | MEDIUM | `patterns-`     |
| 4        | React 19 API           | MEDIUM | `react19-`      |

## 빠른 참고

### 1. 컴포넌트 아키텍처 (HIGH)

- `architecture-avoid-boolean-props` - 동작을 커스터마이즈하기 위해 불리언 prop을
  추가하지 말고, 컴포지션을 사용
- `architecture-compound-components` - 공유 컨텍스트로 복잡한 컴포넌트를 구조화

### 2. 상태 관리 (MEDIUM)

- `state-decouple-implementation` - 상태가 어떻게 관리되는지는 프로바이더만 알고
  있어야 함
- `state-context-interface` - 의존성 주입을 위한 state, actions, meta를 갖는
  제네릭 인터페이스 정의
- `state-lift-state` - 형제 컴포넌트에서 접근할 수 있도록 상태를 프로바이더
  컴포넌트로 이동

### 3. 구현 패턴 (MEDIUM)

- `patterns-explicit-variants` - 불리언 모드 대신 명시적 변형 컴포넌트를 생성
- `patterns-children-over-render-props` - renderX prop 대신 children을 사용해
  컴포지션 구성

### 4. React 19 API (MEDIUM)

> **⚠️ React 19+ 전용.** React 18 이하를 사용한다면 이 섹션을 건너뛰세요.

- `react19-no-forwardref` - `forwardRef`를 쓰지 말고 `useContext()` 대신 `use()`를 사용

## 사용 방법

자세한 설명과 코드 예제는 각 규칙 파일을 확인하세요:

```
rules/architecture-avoid-boolean-props.md
rules/state-context-interface.md
```

각 규칙 파일에는 다음이 포함됩니다:

- 왜 중요한지에 대한 간단한 설명
- 잘못된 코드 예제와 설명
- 올바른 코드 예제와 설명
- 추가 맥락과 참고 자료

## 전체 문서

모든 규칙을 확장한 전체 가이드는 `AGENTS.md`를 참고하세요.
