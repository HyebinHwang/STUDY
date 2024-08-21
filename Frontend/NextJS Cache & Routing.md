# Cache

NextJS에서는 앱 성능을 향상시키기 위해 캐싱을 사용하고 있습니다. 캐싱은 크게 4가지로 구성되어 있습니다. 

### Request Memoization

NextJS는 fetch API를 확장하여 자동적으로 같은 URL과 같은 options를 가지고는 request를 memoize합니다. 리액트 컴포넌트 트리의 여러 위치에서 동일한 데이터를 불러오는 함수를 한 번만 실행해서 호출할 수 있습니다.

예를 들어 레이아웃 컴포넌트와 현재 렌더링 되어야 하는 컴포넌트에서 동일한 데이터를 가져와야 될 때 props를 통해 해당 데이터를 내려주는 것이 아니라 레이아웃 컴포넌트에서 데이터를 가져오고, 그 다음 컴포넌트에서 해당 데이터를 가져오는 코드를 작성해도 데이터를 새로 요청하는 것이 아니라 캐싱된 데이터를 사용합니다.

Request memoization은 Get 요청에서만 사용되고 NextJS의 특징이 아니라 React 특징입니다. 

### Data Cache

NextJS는 server request와 deployments에서 오는 데이터 패칭에 결과들을 지속하는 `Data-Cache`가 빌트인 되어 있습니다.
HTTP cache와 상호작용을 하는 브라우저와 달리, `NextJS`에서의 cache는 server Data Cache와 상호작용하는 server-side request를 가르킵니다. `NextJS` 확장한 `fetch`에 option를 설정하고, 성공적으로 데이터를 가져왔다면 그 응답값을 저장해두고, 동일한 경로로 `fetch`함수를 실행할 때 실제 API호출은 건너뛰고 저장한 응답값을 반환합니다.

### Full Route Cache

NextJS는 자동적으로 빌드타임에 경로를 렌더링하고 캐시합니다. `Full Route Cache`를 이해하려면 리액트가 렌더링을 다루는지 어떻게 NextJS가 결과들을 캐쉬하는지 확인하면 이해하기 수월합니다.

 **1. 서버에서 리액트 렌더링**

렌더링 작업은 각각의 루트 세그먼트와 `Suspense Bounderies`로 나눠져있는 청크로 이루어져 있습니다.
각각의 청크는 두 가지 스텝으로 렌더링 됩니다.

1. 스트리밍을 위해 최적화된 RSC Payload로 데이터 포멧으로 렌더합니다.
2. 서버에서 서버 컴포넌트와 클라이언트 컴포넌트를 사용하여 HTML를 렌더합니다.

즉 작업을 캐싱하거나 응답을 보내기 전에 모든 렌더링이 완료될 때까지 기다릴 필요가 없습니다. 대신에 우리는 작업이 완료되는 결과를 스트리밍 할 수 있습니다. (chunk로 나눠져 있기 때문에)

 **2.서버에서 NextJS 캐싱(`Full Route Cache`)**

NextJS의 기본적인 행동은 서버에서 해당 루트에 렌더된 결과물(RSC Payload and HTM) 을 캐쉬합니다. 이는 빌드 시 또는 재검증 중에 정적으로 렌더링된 경로에 적용됩니다.
루트가 캐싱되는지 혹은 캐싱이 되지 않는지는 해당 경로가 정적인지 다이나믹한지 결정되는 build타임에 선택됩니다. 정적 루트는 기본적으로 캐쉬되지만, Dynamic routes는 request time에 렌더되며 캐쉬되지 않습니다. 

 **3. 클라이언트에서 리액트 하이드레이션과 재조정**
클라이언트에서 request 타임에는, 

1. HTML는 클라이언트 및 서버 컴포넌트의 빠른 인터랙티브 하지 않는 미리보기를 즉시 표시하는데 사용됩니다.
2. RSC Payload는 렌더된 서버 컴포넌트 트리와 클라이언트 재조정 하는데 사용되고, DOM을 업데이트 합니다.
3. 자바스크립트 명령어들은 클라이언트 컴포넌트를 hydrate하는데 사용되고, 앱을 인터랙티브하게 만들어줍니다.

**4. 클라이언트에서 Next.js 캐싱(Router Cache)**
RSC Payload는 클라이언트 사이드안에 개별 경로 세그먼트로 분할된 별도의 인메모리 캐시 Router Cache에 저장됩니다. 해당 Router Cache는 이전에 방문한 경로를 저장하고 향후 경로를 미리 가져와 내비게이션 환경을 개선하는데 사용됩니다.

**5. 후속 탐색(Subsequent Navigations)**
후속 탐색과 prefething 동안에 `NextJS`는 RSC Payload가 Router Cache에 저장되어 있는지 확인합니다. 그렇다면 서버에 새 요청을 보내는 것을 건너뜁니다.
만약에 경로 세그먼트가 캐쉬되어 있지 않다면, `NextJS`는 서버로부터 RSC Payload를 패칭하고, 클라이언트의 라우터 캐시를 채웁니다.

### Client-Side Router Cache

NextJS는 로딩 상태 및 페이지별로 분할된 루트 세그먼트의 RSC 페이로드를 저장하는 in-memory client side 캐싱을 가지고 있습니다.

사용자가 경로 사이를 탐색할 때 NextJS는 방문한 경로 세그먼트를 캐시하고 사용자가 탐색할 가능성이 높은 경로를 prefreching합니다. 따라서 즉시 뒤로/앞으로 탐색하고, 탐색 간에 전체 페이지를 다시 로드하지 않으며 React 상태와 브라우저 상태를 보존할 수 있습니다.

Router Cache의 역할은 다음과 같습니다.

- Layouts은 네비게이션에 재사용되고 캐쉬됩니다.(partial rendering)
- Loading States는 instant navigation을 위해 네비게이션에 재사용되고 캐쉬됩니다.
- Pages는 기본적으로 캐쉬되지 않으며, 브라우저가 뒤로가기 또는 앞으로 가기 하는 동안에는 재사용됩니다. 또한 실험적인 `staleTimes`을 사용하여 페이지 세그먼트를 캐싱할 수 있습니다.

Duration
캐시는 브라우저의 임시 메모리에 저장됩니다. 라우터 캐시의 지속 시간은 두 가지 요인에 의해 결정됩니다.

- session: 캐쉬는 네비게이션 전반에 걸쳐 지속됩니다. 그러나 페이지가 리프레쉬되면 사라집니다.
- 자동적이나 무효화 기간: 레이아웃과 로딩 상태 캐시들은 시간이 지나면 자동적으로 무효화됩니다. 해당 시간은 어떻게 리소스들이 prefetching되었는지, 그리고 자원들이 정적으로 생성되었는지에 따라 결정됩니다.

---

# Routing

클라이언트에서 NextJS는 경로 세그먼트를 `prefetch`를 하고, cache한다. 즉, 유저가 새로운 경로에 네비게이션 할 때, 브라우저는 새로운 페이지를 로드하지 않고, 변경되는 경로 세그먼트만 다시 렌더링하며 네비게이션 경험과 성능을 향상시킵니다.

### Code Splitting

Code splitting을 사용하면 앱 코드를 더 작은 번들로 분할하여 브라우저에서 다운로드하고 실행할 수 있습니다. 이렇게 하면 각 요청에 대한 데이터 전송량과 실행 시간이 줄어들어 성능이 향상됩니다.

서버컴포넌트는 루트 세그먼트에 의해 자동적으로 Code Splitting이 일어납니다. 즉, 네비게이션에 로드되는 현재 경로에 필요로 하는 코드만 필요합니다.

### Prefetching

Prefetching은 유저가 방문하기 전에 백그라운드에서 루트를 preload하는 방법입니다.
NextJS 에서 Prefetching하는 방법은 크게 두 가지 입니다.

- `Link Component`: 경로들은 사용자의 뷰포트에 표시되는 대로 자동으로 prefetch됩니다. Prefetching은 페이지가 처음 로드될 떄 또는 스크롤을 통해 시야에 들어올 떄 발생합니다.
- `router.prefetch()`: useRouter hooks은 경로들을 prefetch하는데 사용될 수 있습니다.

Link의 default prefetching은 loading.js에 따라 달라집니다. 렌더링된 컴포넌트의 트리에서 첫 번째 Loading.js 파일까지 공유 레이아웃만 30초동안 프리페치 및 캐시됩니다. 이렇게 하면 전체 동적 경로를 불러오는데 드는 비용이 줄어들고 로딩 상태를 즉시 표시하여 사용자에게 더 나은 시각적 피드백을 제공할 수 있습니다.
또한 Prefetching은 development에서는 동작하지 않고, Production에서만 동작합니다.

### Caching

NextJS는 Router Cache로 불리는 in-memory client-side cache를 가지고 있습니다. 사용자가 앱을 navigate할 때, RSC Payload는 prefetch된 경로 세그먼트와 방문한 경로들을 캐쉬에 저장합니다.
즉 네비게이션 시 서버에 새로운 요청을 하는 캐시를 재사용하여 요청 및 데이터 전송 횟수를 줄여 성능을 개선합니다.

### Partial Rendering

Partial Rendering은 네비게이션에서 변경된 경로 세그먼트만 클라이언트에서 다시 렌더링되고 공유 세그먼트는 유지된다는 의미입니다.

예를 들어 두 개의 하위 요소인 /dashboard/setting 그리고 /dashboard/analytic 네비게이션할 때, the settings와 analytics 페이지들은 리렌더되며, 공유된 dashboard 레이아웃은 보존됩니다.

### Soft Navigation

브라우저들은 페이지 사이에 네비게이션을 할 때  "hard navigation"을 수행한다. NextJS 앱 라우터는 페이지가 이동할 때 "soft navigation"이 가능하며, 변경된 경로 세그먼트만 다시 렌더링되록 합니다. 이것은 네비게이션 하는 동안 React 상태가 보존되는 것을 가능하게 합니다.

### Back and Forword Navigation

자동적으로, NextJS는 뒤로가기/앞으로가기 네비게이션 때 스크롤 포지션을 저장하고, Router Cache에 루트 세그먼트를 재사용합니다.

---

### 확인해보기

실제 환경에서 prefetching과 router cache를 확인해보기 위해 간단한 코드를 작성해서 확인해보자

```jsx
// /app/good/page.tsx
"use client";

import Link from "next/link";

export default function Good() {
  return (
    <div className="ml-4">
      <div className="flex flex-col">
        <Link className="underline" href="/good/hi">
          HI
        </Link>
        <Link className="underline" href="/good/hello">
          HELLO
        </Link>
      </div>
    </div>
  );
}

// app/good/hi/page.tsx
import Link from "next/link";

export default function Hi() {
  return (
    <div>
      <Link href="/good">되돌아가기</Link>
      <h1>HI</h1>
      <p>
        Lorem ipsum dolor, sit amet consectetur adipisicing elit. Repellendus
        tenetur officia, quaerat modi quod odit tempore aspernatur recusandae
        laudantium, enim officiis optio itaque, explicabo totam fuga assumenda
        quas at aperiam.
      </p>
    </div>
  );
}

// app/good/hello/page.tsx
"use client";

import Link from "next/link";

export default function Hello() {
  return (
    <div>
      <Link href="/good">되돌아가기</Link>
      <h1>HELLO</h1>
      <p>
        APSDOAPSDOPASDOPASDOPASDALSDLKASDJLKASHDKLASDJLKASJDKLASJDKLASJDLKASDJKLASJDKLASJD
      </p>
    </div>
  );
}
```

상위 페이지에 있는 `Good` 컴포넌트는 하위 페이지에 있는 `Hello`와 `Hi` 페이지로 이동시키는 `Link` 컴포넌트를 가지고 있다. `Link` 컴포넌트는 viewport에 들어올 때 컴포넌트를 prefetching을 한다. 그럼 클라이언트 컴포넌트와 서버 컴포넌트는 어떤 식으로 prefetching이 되는지 확인해보자.

`/good`페이지에서 서버컴포넌트인  `HI` Link가 뷰포트에 들어오게 되면, 해당 페이지 Network에는 `Hi`관련 RSC가 추가된다. 

```jsx
2:I[231,["231","static/chunks/231-a667814d58dcbeb4.js","896","static/chunks/app/good/hi/page-4d028e7c71443c0b.js"],""]
3:I[9275,[],""]
4:I[1343,[],""]
0:["nTtSXJ_9IHhccgdEVIrpU",[[["",{"children":["good",{"children":["hi",{"children":["__PAGE__",{}]}]}]},"$undefined","$undefined",true],["",{"children":["good",{"children":["hi",{"children":["__PAGE__",{},[["$L1",["$","div",null,{"children":[["$","$L2",null,{"href":"/good","children":"ë˜ëŒì•„ê°€ê¸°"}],["$","h1",null,{"children":"HI"}],["$","p",null,{"children":"Lorem ipsum dolor, sit amet consectetur adipisicing elit. Repellendus tenetur officia, quaerat modi quod odit tempore aspernatur recusandae laudantium, enim officiis optio itaque, explicabo totam fuga assumenda quas at aperiam."}]]}]],null],null]},["$","$L3",null,{"parallelRouterKey":"children","segmentPath":["children","good","children","hi","children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L4",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":"$undefined","notFoundStyles":"$undefined","styles":null}],null]},["$","$L3",null,{"parallelRouterKey":"children","segmentPath":["children","good","children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L4",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":"$undefined","notFoundStyles":"$undefined","styles":null}],null]},[["$","html",null,{"lang":"en","children":["$","body",null,{"className":"__className_36bd41","children":["$","$L3",null,{"parallelRouterKey":"children","segmentPath":["children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L4",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":[["$","title",null,{"children":"404: This page could not be found."}],["$","div",null,{"style":{"fontFamily":"system-ui,\"Segoe UI\",Roboto,Helvetica,Arial,sans-serif,\"Apple Color Emoji\",\"Segoe UI Emoji\"","height":"100vh","textAlign":"center","display":"flex","flexDirection":"column","alignItems":"center","justifyContent":"center"},"children":["$","div",null,{"children":[["$","style",null,{"dangerouslySetInnerHTML":{"__html":"body{color:#000;background:#fff;margin:0}.next-error-h1{border-right:1px solid rgba(0,0,0,.3)}@media (prefers-color-scheme:dark){body{color:#fff;background:#000}.next-error-h1{border-right:1px solid rgba(255,255,255,.3)}}"}}],["$","h1",null,{"className":"next-error-h1","style":{"display":"inline-block","margin":"0 20px 0 0","padding":"0 23px 0 0","fontSize":24,"fontWeight":500,"verticalAlign":"top","lineHeight":"49px"},"children":"404"}],["$","div",null,{"style":{"display":"inline-block"},"children":["$","h2",null,{"style":{"fontSize":14,"fontWeight":400,"lineHeight":"49px","margin":0},"children":"This page could not be found."}]}]]}]}]],"notFoundStyles":[],"styles":null}]}]}],null],null],[[["$","link","0",{"rel":"stylesheet","href":"/_next/static/css/92ebb0874b823301.css","precedence":"next","crossOrigin":"$undefined"}]],[null,"$L5"]]]]]
5:[["$","meta","0",{"name":"viewport","content":"width=device-width, initial-scale=1"}],["$","meta","1",{"charSet":"utf-8"}],["$","title","2",{"children":"Create Next App"}],["$","meta","3",{"name":"description","content":"Generated by create next app"}],["$","link","4",{"rel":"icon","href":"/favicon.ico","type":"image/x-icon","sizes":"16x16"}],["$","meta","5",{"name":"next-size-adjust"}]]
1:null
```

클라이언트 컴포넌트인 Hello Link가 뷰포트에 들어가게 되면 똑같이 RSC가 추가된다.

```jsx
2:I[6513,[],"ClientPageRoot"]
3:I[1256,["231","static/chunks/231-a667814d58dcbeb4.js","356","static/chunks/app/good/hello/page-f4a6d9f65b5406cc.js"],"default"]
4:I[9275,[],""]
5:I[1343,[],""]
0:["nTtSXJ_9IHhccgdEVIrpU",[[["",{"children":["good",{"children":["hello",{"children":["__PAGE__",{}]}]}]},"$undefined","$undefined",true],["",{"children":["good",{"children":["hello",{"children":["__PAGE__",{},[["$L1",["$","$L2",null,{"props":{"params":{},"searchParams":{}},"Component":"$3"}]],null],null]},["$","$L4",null,{"parallelRouterKey":"children","segmentPath":["children","good","children","hello","children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L5",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":"$undefined","notFoundStyles":"$undefined","styles":null}],null]},["$","$L4",null,{"parallelRouterKey":"children","segmentPath":["children","good","children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L5",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":"$undefined","notFoundStyles":"$undefined","styles":null}],null]},[["$","html",null,{"lang":"en","children":["$","body",null,{"className":"__className_36bd41","children":["$","$L4",null,{"parallelRouterKey":"children","segmentPath":["children"],"error":"$undefined","errorStyles":"$undefined","errorScripts":"$undefined","template":["$","$L5",null,{}],"templateStyles":"$undefined","templateScripts":"$undefined","notFound":[["$","title",null,{"children":"404: This page could not be found."}],["$","div",null,{"style":{"fontFamily":"system-ui,\"Segoe UI\",Roboto,Helvetica,Arial,sans-serif,\"Apple Color Emoji\",\"Segoe UI Emoji\"","height":"100vh","textAlign":"center","display":"flex","flexDirection":"column","alignItems":"center","justifyContent":"center"},"children":["$","div",null,{"children":[["$","style",null,{"dangerouslySetInnerHTML":{"__html":"body{color:#000;background:#fff;margin:0}.next-error-h1{border-right:1px solid rgba(0,0,0,.3)}@media (prefers-color-scheme:dark){body{color:#fff;background:#000}.next-error-h1{border-right:1px solid rgba(255,255,255,.3)}}"}}],["$","h1",null,{"className":"next-error-h1","style":{"display":"inline-block","margin":"0 20px 0 0","padding":"0 23px 0 0","fontSize":24,"fontWeight":500,"verticalAlign":"top","lineHeight":"49px"},"children":"404"}],["$","div",null,{"style":{"display":"inline-block"},"children":["$","h2",null,{"style":{"fontSize":14,"fontWeight":400,"lineHeight":"49px","margin":0},"children":"This page could not be found."}]}]]}]}]],"notFoundStyles":[],"styles":null}]}]}],null],null],[[["$","link","0",{"rel":"stylesheet","href":"/_next/static/css/92ebb0874b823301.css","precedence":"next","crossOrigin":"$undefined"}]],[null,"$L6"]]]]]
6:[["$","meta","0",{"name":"viewport","content":"width=device-width, initial-scale=1"}],["$","meta","1",{"charSet":"utf-8"}],["$","title","2",{"children":"Create Next App"}],["$","meta","3",{"name":"description","content":"Generated by create next app"}],["$","link","4",{"rel":"icon","href":"/favicon.ico","type":"image/x-icon","sizes":"16x16"}],["$","meta","5",{"name":"next-size-adjust"}]]
1:null

```

다른 점은 클라이언트 컴포넌트의 경우 Router Cache로 불리는 in-memory cache를 가지고 있기 때문에, 번들링된 `Hello` 컴포넌트가 `memory cache` 로 저장되어 있는 것을 확인할 수 있다.

<img width="1728" alt="스크린샷 2024-08-21 오후 8 45 40" src="https://github.com/user-attachments/assets/e9ac38ef-9897-4d44-a646-196859fdb38e">

```jsx
(self.webpackChunk_N_E = self.webpackChunk_N_E || []).push([[356], {
    4398: function(n, e, r) {
        Promise.resolve().then(r.bind(r, 1256))
    },
    7138: function(n, e, r) {
        "use strict";
        r.d(e, {
            default: function() {
                return u.a
            }
        });
        var t = r(231)
          , u = r.n(t)
    },
    1256: function(n, e, r) {
        "use strict";
        r.r(e),
        r.d(e, {
            default: function() {
                return i
            }
        });
        var t = r(7437)
          , u = r(7138);
        function i() {
            return (0,
            t.jsxs)("div", {
                children: [(0,
                t.jsx)(u.default, {
                    href: "/good",
                    children: "되돌아가기"
                }), (0,
                t.jsx)("h1", {
                    children: "HELLO"
                }), (0,
                t.jsx)("p", {
                    children: "APSDOAPSDOPASDOPASDOPASDALSDLKASDJLKASHDKLASDJLKASJDKLASJDKLASJDLKASDJKLASJDKLASJD"
                })]
            })
        }
    }
}, function(n) {
    n.O(0, [231, 971, 23, 744], function() {
        return n(n.s = 4398)
    }),
    _N_E = n.O()
}
]);

```
