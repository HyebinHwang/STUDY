# Jest Svg 컴포넌트 에러

NextJS 환경에서 Jest 테스팅 라이브러리를 테스트하고 있었는데 아래와 같은 에러 때문에, 테스트가 되지 않았고 에러를 찾아가는 과정에 대한 글입니다.

<img width="1002" alt="스크린샷 2024-08-25 오후 6 21 29" src="https://github.com/user-attachments/assets/64ad1415-1ecf-4958-98ec-549c35880516">

---

Jest는 환경 설정에서 기본적으로  `testEnvironment`는 `node`로 되어 있으나, 웹앱을 테스트하기 위해 `jsdom`환경을 자주 사용한다.

즉, NextJS에서 코드를 작성하면, 해당 코드를 바로 테스팅을 하는 것이 아니라, JS코드로 트랜스파일링이 된 후에 테스팅을 진행한다. 테스팅을 하려는 코드의 대부분을 생략하고 문제가 있던 부분만 확인해보면, 

```jsx
import { Todo } from "@/types/todo";
import {
  CheckedIcon,
  NotCheckedIcon,
  PenIcon,
  TrashIcon,
} from "../../public/icons";
import { KeyboardEvent, useState } from "react";

interface TodoListProps {
  todo: Todo;
  onClickCheckUpdateBtn: (id: number) => void;
  onClickContentUpdateBtn: (id: number, content: string) => void;
  onClickDeleteBtn: (id: number) => void;
}

export default function TodoList({
  todo,
  onClickCheckUpdateBtn,
  onClickContentUpdateBtn,
  onClickDeleteBtn,
}: TodoListProps) {
	(..)
	(..) 
  return (
    <li className="flex mb-2 border rounded-md px-2 py-1">
      <button
        className="border-none"
        onClick={() => onClickCheckUpdateBtn(todo.id)}
      >
        {todo.isChecked ? (
          <CheckedIcon className="w-5" /> // 확인
        ) : (
          <NotCheckedIcon className="w-5" /> // 확인
        )}
      </button>
      (..)
      (..)
    </li>
  );
}

```

`<CheckedIcon>`과 `<NotCheckedIcon>`이라는 svg확장자의 이미지를 `svgr/webpack`를 이용해 컴포넌트 형식으로 가져와서 사용하고 있다. 

그리고 Jest가 테스트를 하기 위해 위 컴포넌트는 다음과 같은 코드로 변환된다. 

```jsx
code: '"use strict";\n' +
    'Object.defineProperty(exports, "__esModule", {\n' +
    '    value: true\n' +
    '});\n' +
    'Object.defineProperty(exports, "default", {\n' +
    '    enumerable: true,\n' +
    '    get: function() {\n' +
    '        return TodoList;\n' +
    '    }\n' +
    '});\n' +
    'const _jsxruntime = require("react/jsx-runtime");\n' +
    'const _icons = require("../../public/icons");\n' +
    'const _react = require("react");\n' +
    'function TodoList({ todo, onClickCheckUpdateBtn, onClickContentUpdateBtn, onClickDeleteBtn }) {\n' +
    '    (..)
    '    (..)    
    '    return /*#__PURE__*/ (0, _jsxruntime.jsxs)("li", {\n' +
    '        className: "flex mb-2 border rounded-md px-2 py-1",\n' +
    '        children: [\n' +
    '            /*#__PURE__*/ (0, _jsxruntime.jsx)("button", {\n' +
    '                className: "border-none",\n' +
    '                onClick: ()=>onClickCheckUpdateBtn(todo.id),\n' +
    '                children: todo.isChecked ? /*#__PURE__*/ (0, _jsxruntime.jsx)(_icons.CheckedIcon, {\n' +
    '                    className: "w-5"\n' +
    '                }) : /*#__PURE__*/ (0, _jsxruntime.jsx)(_icons.NotCheckedIcon, {\n' +
    '                    className: "w-5"\n' +
    '                })\n' +
    '            }),\n' +
    '            updating ? /*#__PURE__*/ (0, _jsxruntime.jsxs)(_jsxruntime.Fragment, {\n' +
    '                children: [\n' +
    '                    /*#__PURE__*/ (0, _jsxruntime.jsx)("input", {\n' +
    '                        className: "mx-1 px-2 rounded-md w-full",\n' +
    '                        value: value,\n' +
    '                        onChange: ({ currentTarget })=>setValue(currentTarget.value),\n' +
    '                        placeholder: "텍스트를 입력해주세요.",\n' +
    '                        onKeyDown: onKeyDownInput\n' +
    '                    }),\n' +
    '                    /*#__PURE__*/ (0, _jsxruntime.jsx)("button", {\n' +
    '                        className: "flex-shrink-0 px-2 rounded-md bg-black text-slate-50 border-none",\n' +
    '                        onClick: onClickTodoContent,\n' +
    '                        children: "확인"\n' +
    '                    })\n' +
    '                ]\n' +
    '            }) : /*#__PURE__*/ (0, _jsxruntime.jsxs)(_jsxruntime.Fragment, {\n' +
    '                children: [\n' +
    '                    /*#__PURE__*/ (0, _jsxruntime.jsx)("p", {\n' +
    '                        className: `w-full px-2 ${todo.isChecked && "line-through"}`,\n' +
    '                        children: todo.content\n' +
    '                    }),\n' +
    '                    /*#__PURE__*/ (0, _jsxruntime.jsx)("button", {\n' +
    '                        className: "border-none px-1",\n' +
    '                        onClick: ()=>setUpdating(true),\n' +
    '                        children: /*#__PURE__*/ (0, _jsxruntime.jsx)(_icons.PenIcon, {\n' +
    '                            className: "w-4"\n' +
    '                        })\n' +
    '                    }),\n' +
    '                    /*#__PURE__*/ (0, _jsxruntime.jsx)("button", {\n' +
    '                        className: "border-none px-1",\n' +
    '                        onClick: ()=>onClickDeleteBtn(todo.id),\n' +
    '                        children: /*#__PURE__*/ (0, _jsxruntime.jsx)(_icons.TrashIcon, {\n' +
    '                            className: "w-4"\n' +
    '                        })\n' +
    '                    })\n' +
    '                ]\n' +
    '            })\n' +
    '        ]\n' +
    '    });\n' +
    '}\n' +
    (..) 
```

여기서 변환된 코드는 중요하지 않으나, JS로 변환된다는 것을 아는 것은 중요하다. 왜냐하면 현재 `svgr/webpack`을 통해 NextJS에서 컴포넌트 형식으로 svg를 가져오는 것은 설정을 했으나, Jest는 해당 정보를 알지 못한다. Jest에게 Svg 컴포넌트를 읽을 수 있게 할 때는 mock데이터를 활용한다. 즉, svg파일을 만나면 가상의 데이터(mock)데이터로 변환시켜서 읽을 수 있도록 도와준다. 하지만 아래의 코드처럼 svg 파일을 트랜스파일링을 했지만, 다른 사람들은 동작한다고 했는데, 나는 동작되지가 않았다.

```jsx
// __mocks__/svgrMock.js
const SvgUrl = () => "svgrurl";
export default SvgUrl;
```

 

이유는 Jest에 기본으로 설정된 NextJS 설정 때문이었다. [NextJS + Jest는 트랜스파일링을 진행할 때 babel이 아닌 SWC](https://nextjs.org/docs/architecture/nextjs-compiler)를 사용하고, 기본 설정된 config는 다음과 같다.

```jsx
export default function nextJest(options: { dir?: string } = {}) {
  return (
    customJestConfig?:
      | Config.InitialProjectOptions
      | (() => Promise<Config.InitialProjectOptions>)
  ) => {
    return async (): Promise<Config.InitialProjectOptions> => {
	     return {
	       (..)
	     moduleNameMapper: {
          '^.+\\.module\\.(css|sass|scss)$':
            require.resolve('./object-proxy.js'),

          '^.+\\.(css|sass|scss)$': require.resolve('./__mocks__/styleMock.js'),

          '^.+\\.(png|jpg|jpeg|gif|webp|avif|ico|bmp)$': require.resolve(
            `./__mocks__/fileMock.js`
          ),

          // svg 트랜스 파일링
          '^.+\\.(svg)$': require.resolve(`./__mocks__/fileMock.js`),

          '@next/font/(.*)': require.resolve('./__mocks__/nextFontMock.js'),
          
          'next/font/(.*)': require.resolve('./__mocks__/nextFontMock.js'),
         
          'server-only': require.resolve('./__mocks__/empty.js'),

          ...(resolvedJestConfig.moduleNameMapper || {}),
	        },
	        (..)
	        (..)
        }
	     (..)
	     (..)
	  }
}
```

대부분 중략을 하고, 우리가 확인해야 될 곳은 `moduleNameMapper`이다. 해당 코드를 확인하면, svg는 `mocks/fileMock.js` 에 입력된 설정을 확인하여 가상 데이터로 변환된다. 해당 코드는 다음과 같다.

```jsx
// node_modules/next/dist/build/jest/__mocks__/fileMock.js

"use strict";
module.exports = {
    src: "/img.jpg",
    height: 40,
    width: 40,
    blurDataURL: "data:image/png;base64,imagedata"
};

if ((typeof exports.default === 'function' || (typeof exports.default === 'object' && exports.default !== null)) && typeof exports.default.__esModule === 'undefined') {
  Object.defineProperty(exports.default, '__esModule', { value: true });
  Object.assign(exports.default, exports);
  module.exports = exports.default;
}
```

해당 코드는 Svg를 가상의 `img.jpg`로 변환하는 것인데, 동작하지 않는다. 그렇기 때문에 customConfig를 추가해서 svg 컴포넌트를 읽어야 한다. 하지만 아래의 코드로 설정할 경우 동작하지 않는다.

```jsx
// jest.config.js
const nextJest = require("next/jest");

const createJestConfig = nextJest({
  dir: "./",
});

const customJestConfig = {
  setupFilesAfterEnv: ["<rootDir>/jest.setup.js"],
  testEnvironment: "jsdom",
  moduleNameMapper: {
    "\\.svg$": "<rootDir>/__mocks__/svg.js", // 확인
  },
};

module.exports = createJestConfig(customJestConfig);
```

이유는 우리가 설정한 `"\\.svg$": "<rootDir>/__mocks__/svg.js"` 코드는 해당 코드 보다 앞에 있는 기본 NextJS Svg 트랜스 파일링 코드로 대체된다. 아래의 코드는 우리가 작성한 Custom Config가 추가 되는 방식이다.  

```jsx
		     (..)
		      // svg 트랜스 파일링
          '^.+\\.(svg)$': require.resolve(`./__mocks__/fileMock.js`),

          '@next/font/(.*)': require.resolve('./__mocks__/nextFontMock.js'),
          
          'next/font/(.*)': require.resolve('./__mocks__/nextFontMock.js'),
         
          'server-only': require.resolve('./__mocks__/empty.js'),

          ...(resolvedJestConfig.moduleNameMapper || {}),
	        },
	        (..)
        }
```

우리가 직접 `customConfig`를 추가할 경우   `'^.+\\.(svg)$': require.resolve(`./__mocks__/fileMock.js`)` 보다 뒤에 위치하기 때문에, 앞에 있는 트랜스 파일링 코드로 우선 동작한다. 해결 방법은 먼저 config를 불러오고 기본 svg 트랜스파일링 코드보다, 더 앞에 위치하게 합니다. 

```jsx
const nextJest = require("next/jest");

const createJestConfig = nextJest({
  dir: "./",
});

/** @type {import('jest').Config} */
const customJestConfig = {
  setupFilesAfterEnv: ["<rootDir>/jest.setup.js"],
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
  },
  testEnvironment: "jsdom",
};

const jestConfigWithOverrides = async (...args) => {
  const fn = createJestConfig(customJestConfig);
  const res = await fn(...args);

  res.moduleNameMapper = {
    "\\.svg": "<rootDir>/__mocks__/svgrMock.js",
    ...res.moduleNameMapper,
  };

  return res;
};

module.exports = jestConfigWithOverrides;
```

해당 코드를 추가해서 기본으로 설정된 NextJS 트랜스파일링 config보다 더 앞에 위치하도록 변경하니, 정상으로 동작했다.
