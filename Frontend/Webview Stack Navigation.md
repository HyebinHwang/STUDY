# Webview Stack Navigation

현재 저희가 개발하고 있는 앱의 구조는 하단의 탭 네비게이션은 React Native로 개발하고, 그 외의 부분들은 Webview(Next)를 이용해서 개발되어 있습니다.

제목과는 다른 내용이지만 탭만 네비게이션으로 구현한 이유는 앱 스토어에 앱을 배포하기 위해서는 애플에 앱 검증을 받아야 하는데, `Webview`로만 이루어진 앱을 보안상의 이유로 지양하고 있다고 해서 `Native`로 구현된 부분이 필요했습니다. 하단의 탭 네비게이션의 경우 웹 브라우저가 아닌 모바일 앱에서 자주 사용되는 UI이기 때문에, `Native`를 이용해서 구현하는게 아무래도 자주 사용하는 UI이므로 구현도 간단하고, 더 앱처럼 보이도록 구현하기 쉽다고 생각했습니다.

탭은 간단하게 구현할 수 있었으나, 문제는 `Stack Navigation` 이었습니다. 밑의 예제처럼 여러 앱에서 링크를 타고 들어가면 상단에 뒤로가기나, 홈으로 이동할 수 있는 네비게이션이 생기게 되는데 이런 형태를 `Stack Navigation`이라고 부릅니다.

[KakaoTalk_Video_2024-03-01-01-24-23.mp4](../videos/stackNavigation1.mp4)

상단의 영상을 보면 게시글을 클릭하면 상단의 네비게이션 바가 상단에 생기는 것을 확인할 수 있습니다. 제가 생각한 웹뷰에서 앱처럼 보이도록 하는 방법은 크게 두 가지입니다.

1. 페이지 이동 간 애니메이션을 통해 `Stack Navigation` 형태로 보이도록 하기.
2. `React Native`에서 어떻게든 처리하기

처음에 생각한 방식은 웹에서 페이지 이동을 앱의 `Stack Navigation` 형태로 보이도록 하는 것입니다. 이유는 아무래도 RN에서 처리를 하기 위해서는 RN에서 웹의 페이지 이동 변경을 감지하든, 통신을 하든 그 후에 처리를 해야 되기 때문에, 1번 방식이 성능적으로 더 좋다고 생각했습니다.

`Framer motion`이라는 애니메이션 라이브러리를 통해 페이지 이동에 애니메이션을 쉽게 줄 수 있었습니다. 테스트를 진행하고 끝내 구현까지 했지만 선택하지 않았습니다. 그 이유는 앱처럼 사이드 스와이프로 이전 페이지로 이동하는 것을 구현하기는 너무 복잡했습니다.

[KakaoTalk_Video_2024-03-01-14-04-23.mp4](../videos/stackNavigation2.mp4)

해당 문제점을 테스트를 진행하면서 알게 된점이 아쉬웠고 다음부터는 테스트 전에 필요한 기능들을 전부 나열하고 고민해보는 단계가 있어야 됐다고 생각했습니다. 그래서 결론적으로 선택한 방식은 2번인 `React Native에서 처리하기` 입니다. 

어떤 방식으로 처리해야 되나 여러 고민을 해봤는데, 떠오르지 않아서 여러 블로그들을 참고했습니다. 주로 참고한 페이지는 [React Native Webview에 Stack Navigation](https://velog.io/@rkd028/React-Native-Webview%EC%97%90-Stack-Navigation-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0) 적용하기 입니다. 해당 방식의 플로우에 대해서 user페이지로 짧게 설명하면 

1. RN에서 `Webview`로 `/users`로 URL 연동
2. `/users` URL로 연결된 웹 노출
3. 앱에서 해당 웹에 링크 클릭 
4. 웹에서 해당 링크를 클릭했다는 메시지를 RN으로 보내기
5. RN에서 `Stack Navigation`처리하기

이제 부터 코드를 살펴보겠습니다. 해당 링크를 클릭했다는 메시지를 **웹에서 RN으로 보내는 코드**를 살펴보겠습니다.

```jsx
export default function NativeLink({ children, className, type = 'link', ...props }: NativeLinkProps) {
  const router = useRouter();
  const onClickLink = (event: MouseEvent<HTMLAnchorElement>) => {
    event.preventDefault();
    const href = props.href as string;
    if (!window.ReactNativeWebView) {     // Webview 체크
      if (type === 'back') {
        router.back();
        return;
      }
      router.push(href);
      return;
    }

    if (type === 'back') { // Webview일 경우 postMessage를 통해 메시지 보내기
      window.ReactNativeWebView.postMessage(JSON.stringify({ type: 'BACK', data: href }));
      return;
    }
    window.ReactNativeWebView.postMessage(JSON.stringify({ type: 'ROUTER', data: href }));
  };
  return (
    <Link {...props} className={className || ''} href={'#'} onClick={onClickLink}>
      {children}
    </Link>
  );
}
```

`window.ReactNativeWebView`를 통해 `Webview`를 체크하고, 웹뷰가 아닐 경우는 `useRouter`를 통해 링크를 이동해주고, 웹뷰일 경우는 `postMessage`로 `href` 즉, 이동되어야 하는 페이지의 `URL`를 보내는 방식을 사용했습니다. 다음으로 **RN에서 메시지를 수신하는 코드**를 살펴보겠습니다.

```jsx

import { StackActions } from "@react-navigation/native";
import { useNavigation } from "expo-router";

export default function Profile(){ 
 const navigation = useNavigation();

 const requestOnMessage = async (event: WebViewMessageEvent) => {
    const nativeEvent = JSON.parse(event.nativeEvent.data);
    if (nativeEvent?.type === "ROUTER") {
      const path: string = nativeEvent.data;
      if (path === "BACK") {
        const popAction = StackActions.pop(1);
        navigation.dispatch(popAction);
      } else {
        const pushAction = StackActions.push("detail", {
          url: nativeEvent.data,
          isStack: true,
        });
        navigation.dispatch(pushAction);
      }
    }
  };
...
... 
<WebView
   onMessage={requestOnMessage}
   source={{ uri: Route.url + "/user" }}
 />
}
```

`Webview`에 `onMessage`라는 기능을 통해 `postMessage`를 수신할 수 있습니다. 링크 이동이 되면 `onMessage`이벤트를 통해 메시지를 수신하고, 웹에서 보낸 `href`를 받아서 `StackActions`과 `navigation`를 통해 RN에 `detail`이라는 파일로 `params`를 추가해서 이동시킵니다.

```jsx
// detail.tsx
import { useLocalSearchParams } from "expo-router";

export default function ProfileDetail() {
  const params = useLocalSearchParams<{ url?: string }>();

  return (
    <>
      <Stack.Screen
        options={{
          headerShown: true,
        }}
      />
      <View style={{ flex: 1 }}>
        <WebView source={{ uri: Route.url + params.url }} />
      </View>
    </>
  );
}
```

`detail.tsx`에서는 `params`를 통해 URL을 가지고, `Webview`에 연동을 하면 `Stack` 형식으로 앱에서 노출되게 됩니다. 

참고: 

[https://sinnerr0.medium.com/reactnative에서-webview를-사용할-때-multi-webview를-사용하며-stack으로-쌓는-방안-6134d466b5bc](https://sinnerr0.medium.com/reactnative%EC%97%90%EC%84%9C-webview%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%A0-%EB%95%8C-multi-webview%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B0-stack%EC%9C%BC%EB%A1%9C-%EC%8C%93%EB%8A%94-%EB%B0%A9%EC%95%88-6134d466b5bc)

[https://velog.io/@rkd028/React-Native-Webview에-Stack-Navigation-적용하기](https://velog.io/@rkd028/React-Native-Webview%EC%97%90-Stack-Navigation-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)
