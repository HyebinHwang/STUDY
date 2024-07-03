# Bun & Elysia

Bun은 최신 자바스크립트 생태계를 고려하여 설계된 새로운 자바스크립트 런타임입니다.

### 자바스크립트 런타임

런타임은 프로그래밍 언어 지금은 **자바스크립트가 실행할 수 있는 환경**을 의미합니다. 흔한 예로 크롬, 사파리들 웹 브라우저가 있습니다. 웹 브라우저는 HTML 문서에서 `script`태그를 통해 삽입된 자바스크립트 코드를 실행하여 웹페이지가 사용자와 상호 작용 할 수 있도록 도와줍니다. 또한 `Node.js`도 런타임이며, 다른 점은 서버처럼 브라우저가 아닌 환경에서 사용할 수 있습니다. 

`Bun`은 `Node.js`처럼 기본적으로 백엔드에서 사용하기 위해 만들어진 자바스크립트 런타임이며, 저수준 언어인 Zig라는 언어를 통해 개발되었습니다.

### `Bun`의 설계 목표

1. Speed

`Bun`은 `Node.js`보다 4배 빠릅니다. 빠른 이유는 `Node.js`는 V8엔진을 사용했지만 `Bun`은 Javascript Core엔진을 사용했기 때문입니다. Javascript Core 엔진은 빠른 시작을 위해 최적화되어 있습니다.

2. Typescript & JSX support

다른 모듈을 다운받지 않고 바로 `.jsx`, `.ts` and `.tsx`파일들을 실행할 수 있습니다. `Bun`의 트랜스파일러는 이러한 파일들을 자바스크립트가 실행되기 전에 변환합니다.

3. ESM & CommonJS compatibility

최신의 자바스크립트 생태계는 ES modules를 사용하는 추세지만, npm의 많은 패키지들은 common JS를 사용하고 있습니다. Bun은 ES modules를 추천하고, CommonJS를 지원합니다.

4. Web-standard APIs

Bun은 `fetch`, `Websoket`, `ReadableStream` 같은 Web API들을 지원합니다. Bun은 Apple에서 Safari용으로 개발한 JavaScriptCore 엔진으로 구동되므로 헤더 및 URL과 같은 일부 API는 Safari의 구현을 직접 사용합니다.

5. Node.js 호환성

Bun은 노드의 drop-in replacement로 설계되었기 때문에 아무것도 변경할 필요없이 바로 노드 앱에서 bun을 실행할 수 있습니다.

Bun의 목표는 런타임 그 이상입니다. 장기적인 목표는 패키지 관리자, 트랜스파일러, 번들러, 스크립트 런처, 테스트 런처 등을 포함하여 JavaScript/TypeScript로 앱을 빌드하기 위한 일관된 인프라 툴킷이 되는 것입니다.

해당 기술을 선택한 이유: 우선 속도도 더 빠루고, Node.js에서 마이그레이션도 편하고, ESM 및 Web APIs들을 지원하기 때문에 프론트엔드가 작업하기 훨씬 수월하기 때문에 선택했습니다.

---

### ElysiaJS

엘리시아는 자바스크립트 또는 타입스크립트로 Bun용 백엔드 서버를 구축하기 위한 인체공학적 웹 프레임워크입니다. 쉽게 말해 Node.js에 `Express`처럼 **서버를 간단하게 구현할 수 있도록 설계된 프레임워크**입니다. 

### `Elysia`의 특징들

다른 프레임워크와 차별화되는 엘리시아의 가장 사랑받는 기능은 다음과 같습니다.

- 성능 - 코드 분석을 통한 최적화된 코드 생성
- 통합 유형(Unified Type)
- End-to-end Type Safety - 클라이언트와 서버 모두에서 데이터 동기화
- Typescript
- 인체공학적 디자인 - 서버 구축을 위한 간단하고 친숙한 API
- JSX Template Engine - 프론트엔드 개발자를 위한 익숙한 환경

### Typescript

`Elysia`의 타입 시스템은 명시적인 타입을 작성할 필요 없이 코드를 자동으로 타입으로 추론하도록 미세 조정되었으며, 런타임과 컴파일 타임 모두에 타입 안전성을 제공하여 가장 인체공학적인 개발자 환경을 제공합니다. 예제를 확인하면, 

```jsx
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id }}) => id)
    .listen(3000)
```

대부분의 프레임 워크에서는 `id`에 타입을 작성해야 되지만, `Elysia`는 `params.id`를 항상 사용할 수 있고 문자열로 입력한다는 것을 이해합니다. `Elysia`의 목표는 유저가 타입스크립트 작성을 줄이고, 비즈니스 로직에 더 집중하는 것입니다. 

### Unified Type

한 걸음 더 나아가, `Elysia`는 런타임과 컴파일 타임 모두에서 유형과 값의 유효성을 검사하여 데이터 유형에 대한 단일 데이터 소스를 생성하는 스키마 빌더인 `Elysia.t`를 제공합니다. `Elysia`는 이 용어를 `Unified Type` 통합 유형이라고 부릅니다.

예제 코드를 string 대신 numeric value만 허용하도록 이전 코드를 수정해 보겠습니다.

```jsx
const examlple = "12345" // Numeric Value
```

```jsx
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id }}) => id, {
        params: t.Object({
            id: t.Numeric()
        })
    })
    .listen(3000)
```

이 코드는 경로 매개변수 `id`가 항상 숫자 문자열(Numeric String)이 되도록 한 다음 런타임과 컴파일 타임(유형 수준) 모두에서 자동으로 숫자로 변환되도록 합니다. `Elysia` 스키마 빌더를 사용하면 신뢰할 수 있는 단일 소스를 통해 강력한 타이핑 언어와 같은 유형 안전성을 보장할 수 있습니다.
