# PNPM의 장점에 대해서

### Boosting installation Speed
pnpm은 의존성을 설치할 때 다음과 같은 과정으로 진행된다.
1. 의존성 해결(resolving): 모든 필요한 의존성들은 스토어에 정의되고 패치된다.
2. 폴더 구조 계산: node_modules 폴더 구성은 의존성들을 기반으로하여 측정된다.
3. 의존성 연결(Linking): 모든 남아있는 의존성들은 패치되고, 스토어에서 node_modules로 하드링크된다.
이러한 방법은 기존의 npm의 three-stage 설치 과정에 비해 더 빠른 속도를 유지할 수 있다.

npm 의존성 three-stage설치 전략
1. Resolving: package.json과 package-lock.json을 확인하여 프로젝트에서 필요한 모든 패키지와 그에 따른 의존성을 찾아내고 어떤 버전을 설치해야 되는지 결정하는 과정
2. Fetching: 패키지를 실제로 다운로드하는 단계 
3. Writing: 다운로드한 패키지를 node_modules폴더에 설치하는 단계
NPM에 설치에서 이러한 단계는 모든 패키지들을 설치할 때 Resolving단계가 끝나게 되면, 그다음 Fetching 마지막으로 Writing순서로 진행되게 된다. 즉, 만약에 어떠한 의존성이 Resolving단계가 먼저 끝나도 다른 패키지들 전부가 Resolving단계가 마무리가 되어야 그 다음 Fetching단계로 진행될 수 있다. 반면에 PNPM은 다른 패키지들을 기다리지 않고, 바로 Resolving단계가 끝나면 Fetching단계로 진행되기 때문에, 설치 속도도 더 빠르다.
### Creating a non-flat node_modules directory
NPM은 3.xx 이후의 버전에서는 flatted dependency tree를 사용하고 있다. flat dependency tree는 의존성을 최상위 폴더에 설치하여, 중복 설치를 줄이고 node_modules폴더의 깊이를 줄이는 방식이다. 예를 들어 A와 B가 같은 의존성 C를 필요로 한다고 가정하면 최상위 폴더에 C를 설치해서 A와 B 각각에서 C를 설치하는 것이 아니라, 상위 폴더에 C를 설치하여 중복을 줄이는 방법이다. 하지만 호환되지 않은 의존성 버전이 있을 경우는 각각의 node_modules에 C의 다른 버전이 설치 될 수도 있다.

하지만 pnpm은 non-flat방식의 폴더 구조로 구성되어 있다. 예를 들어 npm과 pnpm에 express를 설치를 한다고 가정해보면, npm의 node_modules는 아래와 같은 폴더 구조를 가지게 된다.
```
.bin  
accepts  
array-flatten  
body-parser  
bytes  
content-disposition  
cookie-signature  
cookie  
debug  
depd  
destroy  
ee-first  
encodeurl  
escape-html  
etag  
express
```
 반면에 pnpm은 아래와 같은 형식을 가지게 된다.
```
.pnpm  
.modules.yaml  
express 
```
express와 관련된 의존성들이 설치된 npm과 달리 pnpm은 .pnpm과 express만 존재한다. 그렇다면 express를 아래의 항목에서 확인하면
```
▾ node_modules  
  ▸ .pnpm  
  ▾ express  
    ▸ lib  
      History.md  
      index.js  
      LICENSE  
      package.json  
      Readme.md  
   .modules.yaml
 ```
express폴더를 보면, 다른 의존성 파일에 대한 정보가 보이지 않는다. 그 이유는 express는 단지 symlink이기 때문이다. Node.js가 의존성 중에 어떤 것을 설치해야 될 지 결정할 때(resolving) 실제 위치를 사용하므로, 심볼릭 링크를 보존하지 않는다. 

Hardlink vs Symlink
- Hardlink: 선택한 파일의 사본역할. 원본 파일이 삭제되더라도 파일에 대한 링크는 여전히 데이터를 포함하고 있어 데이터에 엑세스할 수 있다.
- Symlink: 파일 이름에 대한 포인터(참조)약할. 원본 파일이 삭제된다면 해당 링크는 더 이상 존재하지 않는다.

실제 위치는 node_modules/.pnpm/express@4.18.1/node_modules/express.에 위치한다. 우리는 해당 폴더를 virtual stores라고 부르고, 모든 flat 폴더 구조에 패키지들이 저장되고 모든 패키지들에 대한 정보를 확인할 수 있다.
또한 express에 대한 node_modules폴더는         node_modules/.pnpm/express@4.18.1/node_modules/express/node_modules에서 확인할 수 있다.

그렇다면 non-flat 폴더 구조의 장점은 무엇일까.

**모듈 격리**
예를 들어 A와 B를 설치하는데 둘 다 C를 의존한다고 가정하면, flat구조에서는 기본적으로 node_modules에  C라는 의존성을 설치한다. 하지만 만약에 A와 B가 다른 버전의 C를 사용할 경우 충돌이 발생할 수 있기 때문에, 각각의 A와B 의존성 폴더에 node_modules가 추가되고 알맞은 버전 C를 설치하게 된다. 하지만 이러한 충돌이 많아질 경우 구조가 복잡해질 수 있다는 단점이 있다. 
반면에 PNPM은 각 패키지가 고유한 node_modules폴더에서 관리하기 때문에 의존성 충돌을 방지할 수 있다.

**하드링크와 심볼릭 링크를 통한 디스크 공간 절약**
PNPM은 중앙 스토리지에 의존성 파일을 한 번만 설치하고, 각 프로젝트의 node_modules폴더에는 하드 링크를 만들에서 연결하는 구조이기 때문에 동일한 의존성을 가진 패키지들이 있더라도 한 번만 설치를 하게 된다. 하지만, NPM같은 경우는 node_modules에 모든 의존성을 설치하고 관리하지만, 동일한 의존성을 사용하는 경우 각 폴더에 node_modules에서 해당 의존성 파일을 설치하기 때문에, 디스크 공간을 더 많이 사용하게 될 가능성이 생기게 된다.

예를 들어 A, B, C패키지들이 모두 lodash@4.17.0을 사용하고, D패키지는 lodash@3.10.0을 사용한다면 NPM의 경우 node_modules에서 lodash@4.17.0을 여러 번 설치될 가능성이 있다. 하지만 PNPM의 경우는 중앙 스토리지에 한 번만 설치되고, 각 프로젝트의 node_modules에는 파일을 가리키는 하드링크만 존재하기 때문에, 재설치를 통한 디스크 공간이 낭비될 가능성이 없다

**더 나은 모노레포 지원**
PNPM은 의존성을 중앙에서 관리하고, 하드링크를 통해 효율적으로 링크를 걸 기 때문에, 더 유리하다.
- NPM은 flat구조는 모노레포에서 의존성 관리가 더 복잡해질 수 있다.
- PNPM은 각 패키지가 고유한 의존성을 가지면서도 효율적인 디스크 공간을 소유하고 있으므로 모노레포에 더 작합하다.

----- 

참고
- https://pnpm.io/motivation
- https://pnpm.io/blog/2020/05/27/flat-node-modules-is-not-the-only-way
- https://pnpm.io/symlinked-node-modules-structure

