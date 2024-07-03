# SSR
해당 글은 NextJS와 Tanstack Query를 이용해 Server Side Rendering를 추가하는 과정에 대한 글입니다.

프로젝트를 2명의 프론트엔드 개발자가 정해진 기간안에 풀스택으로 진행하다 보니 프로 백엔드보다 데이터 응답 속도가 느린 편인 경우가 있었습니다. 그리고 또한 유저가 화면에 진입했을 때 빈 페이지를 보여주고, 그 다음에 데이터를 채우는 방식보다 유저 첫 진입부터 데이터를 보여주는 것이 더 좋은 유저 경험을 제공하기 때문에 SSR 적용을 고민했습니다. 

적용 하기에 앞서 궁금한 부분은 소문만 무성한 React Server Component를 적용해야 되나, 아니면 NextJS에 SSR를 적용 해야 되나 아니면 이 두개가 동일한 것인가에 대한 개념이 정리되어 있지 않았습니다.

## SSR

SSR은 좋은 글이 많기 때문에 간단하게 설명하면 SSR은 서버에서 사용자에게 보여줄 페이지를 미리 구성하여 사용자에게 페이지를 보여주는 방식입니다. SSR과 대척점에 있는 CSR과의 차이점은 SSR방식의 서버는 즉시 렌더링 될 수 있는 HTML을 응답하고, CSR은 JS링크가 포함된 빈 문서를 가져오게 됩니다.

그렇기 때문에, 장점으로는 

1. SSR은 초기 페이지를 제공하므로, 사용자 경험이 좋은 편.
2.  텅빈 HTML을 넘겨주는 CSR보다 SEO가 향상 

단점으로는

1. 반면에 단점은 웹의 성능을 결정하는 TTFB(Time To First Byte)가 CSR보다 느림.
2. 유저가 페이지를 더 빨리 볼 수 있지만 JS가 아직 연결되지 않을 경우 유저와의 상호작용이 발생하지 않을 수 있음.

> TTFB(Time To First Byte): 수신하는 페이지의 첫 번쨰 바이트까지 HTTP요청을 하는 기간 즉, 클라이언트 요청 + 서버의 요청 처리 시간 + 서버의 클라이언트로 응답 = TTFB
> 

---

## RSC(React Server Component)

React Server Component는 미리 실행되고 JS 번들에서 제외되는 새로운 종류의 컴포넌트입니다.

React 공홈에서의 RSC 특징을 확인하면 

1. 미리 실행되고 자바스크립트 번들에서 제외되는 컴포넌트 
2. 빌드 중에 실행되어 파일 시스템을 읽거나 정적 콘텐츠를 가져올 수 있음.
3. 서버에서 실행할 수 있으므로, 데이터 레이어에 엑세스 할 수 있음.
4. MPA(Multi Page App)과 SPA(Single Page App)를 결합하여 두 가지 장점을 모두 제공합니다. 

그리고 NextJS에서는 RSC를 이용해서 캐싱 및 Streaming이라고 해서 렌더링 작업을 청크로 분할하여 준비되는 대로 클라이언트로 스트리밍 할 수 있는 기능을 지원합니다.

RSC의 렌더링되는 방식은 다음과 같습니다.

1. React Server Component Payload라는 데이터 포멧으로 Server Component를 렌더합니다.
2. NextJS는 RSC payload 및 클라이언트 컴포넌트 자바스크립트 명령어를 사용하여 서버에서 HTML를 렌더링합니다.

그리고 나서 클라이언트에서는

1. HTML은 페이지의 non-interacetive한 항목들을 즉시 보여줍니다.
2. RSC payload는 클라이언트 및 서버 컴포넌트 트리를 조정하고 DOM 을 업데이트하는데 사용됩니다.
3. 자바스크립트는 클라이언트 컴포넌트에 hydrate를 하고, 애플리케이션을 interactive하게 만드는데 사용된다.

> React Server Component Payload는 서버컴포넌트의 압축된 바이너리 표현이다.
> 

---

### 어떤 것을 적용..?

현재 프로젝트에서 필요한 기능인 유저가 해당 페이지에 진입하기 전에 미리 데이터를 가져오는 것을 RSC를 사용해야 되는지 SSR를 사용해야 되는지 둘이 동일한 방식인지는 아직 확신이 들지는 않았습니다.

적용하기에 앞서 Tanstack Query를 통해 두 개념에 차이를 조금이나마 이해할 수 있었습니다. 해당 프로젝트에 필요한 기능은 **prefetchQuery**인데, 해당 기능이 NextJS에 **getServerSideProps/getStaticProps**와 유사하다고 되어 있습니다. 

NextJS에 해당 기술에 대한 설명을 확인하면 Pre-rendering 방식으로 되어 있습니다. Pre-rendering 방식은 사전에 렌더링하는 방식으로 크게 두 가지 형태를 가지고 있습니다. 

1. SSG: 빌드 타임에 생성되고 모든 요청마다 재사용
2. SSR: 각 요청마다 생성

즉, RSC는 새로운 서버에서 실행할 수 있는 Component에 대한 설명이고, 해당 프로젝트에 필요한 기능은 **Pre-rendering**입니다. 프로젝트에 적용되어 있는 route방식은 RSC가 빌트되어 있는 [App router](https://nextjs.org/docs/app/building-your-application/routing) 방식이기 때문에 **정리하면** **RSC를 이용해서 SSR Pre-rendering를 사용하는 것이라고 생각합니다.**

---

## Tanstack Query를 이용해 Pre-rendering적용

자세한 코드는 [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr)에 설명이 자세히 나와있습니다.

```jsx
// 기존의 Client Side 전용 Query Client를 Server Side에서 적용되도록 수정
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactNode } from 'react';

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000,
      },
    },
  });
}

let browserQueryClient: QueryClient | undefined;

function getQueryClient() {
  if (typeof window === 'undefined') {
    return makeQueryClient();
  }
  if (!browserQueryClient) browserQueryClient = makeQueryClient();
  return browserQueryClient;
}

export default function Providers({ children }: { children: ReactNode }) {
  const queryClient = getQueryClient();

  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}

```

```jsx
import { HydrationBoundary, QueryClient, dehydrate } from '@tanstack/react-query';
import { cookies } from 'next/headers';

export default async function ChatRoomPage({ searchParams: { postId }, params: { id } }: DynamicRouteParams) {
  const queryClient = new QueryClient();
  const cookieStore = cookies();
  const token = cookieStore.get('token');

  await queryClient.prefetchQuery({
    queryKey: [MESSAGE_POST_KEY, postId],
    queryFn: () => getPostPetDetail(postId, token?.value),
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <ChatRoom postId={postId} id={id} />
    </HydrationBoundary>
  );
}
```

`HydrationBoundary`를 통해 클라이언트 `queryClient`에 이전에 `dehydrated`되어 있는 `queryClient`를 `hydrate`합니다. 클라이언트에 이미 데이터가 포함되어 있는 경우 업데이트 타임스탬프에 따라 새 쿼리가 지능적으로 병합됩니다. `searchParams`나 `cookie`같은 경우는 `NextJS` 내장 라이브러리를 통해 정보를 가져올 수 있습니다.

```jsx
export default function ChatRoom({ id, postId }: ChatRoomProps) {
	...
	... 
	... 
  const { data: post } = useQuery<Post>({
    queryKey: [MESSAGE_POST_KEY, postId],
    queryFn: () => getPostPetDetail(postId),
  });
  
   ...
   ... 
   ... 
  }
```

prefetchQuery와 같은 QueryKey를 가진 데이터의 경우는 유저가 접속하면 mounted된 이후가 아닌 바로 해당 데이터를 확인할 수 있습니다.  

---

결론

너무 큰 데이터의 경우 TTFB가 문제이긴 하지만, SEO 및 유저 경험이 좋기 때문에 엄청나게 큰 데이터를 받는 것이 아니면 Client Component와 Server Component를 혼합하여 사용하면 더 좋은 경험을 제공할 수 있을 것 같다.

참고

https://d2.naver.com/helloworld/7804182 

[https://nextjs.org/docs/app/](https://nextjs.org/docs/app/building-your-application/rendering/server-components)

https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components

https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr
