# yarn berry pnp

## yarn berry 
[공식 github](https://github.com/yarnpkg/berry)

`yarn berry` 는 기존 `yarn classic` 의 업그레이드 버전으로 , 기존 사용되는 것과는 다른 새로운 패키지 관리자입니다. 기존에 가지고 있던 여러 문제점들을 해결하고 , 새롭게 가능해지는 여러 기능들이 추가되었습니다. 
- 여러 플러그인들이 추가되었습니다. [공식 플러그인 문서](https://github.com/yarnpkg/berry?tab=readme-ov-file#yarn-plugins)
- 기존 `node_modules` 가 가지고 있던 모듈 탐색 과정의 비효율성, ghost dependency 문제를 pnp를 통해서 해결할 수 있습니다. 
  - [모듈 탐색 비효율성](https://yarnpkg.com/features/pnp)
  - ghost dependency : [hoisting](https://yarnpkg.com/advanced/lexicon#hoisting) 으로 인해 직접 설치하지 않았지만 , 다른 라이브러리를 설치하면서 같이 설치되는 dependency 
  - [추가 참고 문서](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/)
- pnp ( [Plug'n'Play](https://yarnpkg.com/features/pnp) ) 를 `node_modules` 대신 사용가능하게 되면서 , 위 문제를 해결하고 dependency 관리를 쉽게 할 수 있도록 도와줍니다.
- zero-install : pnp 를 도입하면서 의존성 관리에 드는 용량이 작아지면서 , `.yarn` 파일을 `git` 에 업로드 하여 `yarn install` 이 필요없어 집니다.

## pnp

패키지 의존성을 node_modules 를 사용하지 않고 , `.yarn` 내부 파일에 저장하고 관리하는 방식으로 , `.pnp.cjs` 파일 안에 어떤 라이브러리에 의존하고 어디에 위치하는지 알 수 있는 방법입니다. 이에 따라서 보다 적은 용량으로 dependency 가 관리되고 , 기존 node_modules가 가지고있던 두 가지 문제점이 해결 가능해집니다.

## 사용 경험

점점 패키지가 커지게 되면서 , 보다 많은 라이브러리가 설치되면서 dependency 의 개수가 증가하고 , `yarn install` 시간도 점점 오래걸리게 되어서 해결방안을 찾는 와중에 도입하게 되었습니다. 기존에는 시작 전에 강박처럼 `yarn install` 을 진행하고 기다리는 시간이 많이 있었는데 , pnp 와 zero-install 도입 후 그 시간이 많이 줄어들었습니다. 다만 처음에 적용하는데 생각보다 정보가 없어서 어려움이 많이 있었는데 , migration 이후에 사용하는데는 크게 어려움이 없었습니다. 또한 빌드시간도 단축되어서 , 6분정도 걸리던 `github action` 이 4분정도로 단축되었던 경험이 있습니다.  

## 적용 과정

### 1. 기본 프로젝트 설치 ( vite )
[vite](https://ko.vitejs.dev/guide/) 홈페이지 참고해서 기본 프로젝트를 생성한다. 

```shell
yarn create vite
```
기본 커맨드를 입력하면 선택창이 나오는데 , 하나씩 원하는 타입을 선택하면 된다. 여기서는 기본적인 `React + Typescript` 를 사용해서 만들었다. 
![](https://velog.velcdn.com/images/k1my3ch4n/post/6827cd68-03cb-494f-8d06-6a1f52d23243/image.png)

### 2. yarn berry 설치
```shell
# yarn berry 설치
yarn set version berry
```
![](https://velog.velcdn.com/images/k1my3ch4n/post/dc0ae202-63ca-4d25-a710-9d1512737395/image.png)*설치완료!*


```shell
# yarn version 확인
yarn --version
```
![](https://velog.velcdn.com/images/k1my3ch4n/post/81dcd9dc-a7f9-4e30-a76d-9f32518da143/image.png)*버전 확인*

설치가 완료되면 아래와 같이 파일이 만들어집니다. ![](https://velog.velcdn.com/images/k1my3ch4n/post/790c8cec-7b13-48bc-9ee7-293f221c639b/image.png)

그 후 , 생성된 `.yarnrc.yml` 파일에 아래와 같이 추가합니다. 

```yml
nodeLinker: pnp
enableScripts: false
```
- `nodeLinker` : `pnp` 와 `node_modules` 중 어떤 모드를 선택할지 알려줌. 기본값은 `pnp`
- `enableScripts` : `zero-install` 을 위해 추가. 다만 `false` 로 추가한다면 , `.gitIgnore` 에 `.yarn/unplugged` 를 추가해 주어야함.

### 3. dependency 설치

위까지 진행한 후 , 의존성을 설치한다.

```shell
yarn install
```

그 후 , 기본 `App.tsx` 파일에 들어가면 .. 
![](https://velog.velcdn.com/images/k1my3ch4n/post/f1b6b473-a44b-4759-9e58-80e45b8dd165/image.png)

위와 같이 `JSX.IntrinsicElements` 에러가 발생한다. 해당 에러는 vscode ~~~~~

[공식문서](https://yarnpkg.com/features/pnp)에서는 아래와 같이 해결하고 있다.

```shell
# vscode 용 플러그안
yarn dlx @yarnpkg/sdks vscode
```

```
ctrl + shift + p 입력 후

select typescript version 입력
```
위 순서를 따르면 아래와 같은 사진이 등장하는데 ,
![](https://velog.velcdn.com/images/k1my3ch4n/post/2e253c5a-89fc-481e-9b33-9f8488ff3c2f/image.png)

2번째 `작업 영역 버전 사용` 을 클릭한다. 그러면
![](https://velog.velcdn.com/images/k1my3ch4n/post/2215c4f0-1bf8-4597-8701-23bc20a40d21/image.png)*에러가 사라졌다 !*

그 후 `yarn dev` 를 시전하면 ..

![](https://velog.velcdn.com/images/k1my3ch4n/post/57330fa0-e515-4c8b-aaa7-3e0e668ad10f/image.png)*짜잔!*

### 후기

사실 이 글에서는 기초적인 설정만 추가해서 그렇게 큰 어려움은 없었지만 , 기존 프로젝트를 migration 하는 과정에서 pnp를 적용할때 여러 dependency 오류가 있었던 기억이 있다. 그중에는 해결하지 못해서 흐린 눈을 하고 숨은 기억도 있는데 ,,, 추후 작업하면서 오류 및 해결과정도 여기에 추가로 업로드할 계획입니다. 

`yarn berry` 의 가장 큰 장점은 `yarn install` `yarn build` 시 굉장한 시간 단축이 된다는 점입니다. 사실 프로젝트가 커지면 커질수록 점점 더 오래 걸리기 마련인데 , 그 시간들을 단축해 준다는건 생산성과도 연결되는 부분이라 더 와닿았던 것 같습니다. 

- 잘못된 부분, 추가해야하는 부분이 있다면 알려주시면 감사하겠습니다 ! 
