## 3장 리액트 훅 깊게 살펴보기

## 3.1 리액트의 모든 훅 파헤치기

클래스형에서만 가능했던 state, ref 등의 기능을 가능하게 만든 훅에 대해 이해하자

### 3.1.1 useState

- 함수형 컴포넌트 내부에서 상태를 정의하고 관리할 수 있게 해줌
- **useState 구현**
    - 아무런 정의 하지않으면 undefined 초깃값
    - 클로저를 이용해서 구현
    
    <aside>
    💡 useState 구현과정
    
    1. 애플리케이션 전체의 states 배열 초기화, 최초접근일때 초기화
    2. state 정보 조회해서 현재 값있는지 확인, 없다면 초기값 설정
    3. state 값을 위에서 조회한 현재 값으로 업데이트
    4. 즉시 실행 함수로 setter 생성
    5. 현재 index를 클로저로 가두어 이후에도 동일한 index에 접근할 수 있도록 함
    6. 컴포넌트 렌더링
    </aside>
    
- **게으른 초기화** : 원시값 이외에 함수를 초기값을로 넣어주는 것 (lazy initialization)
    - 초깃값이 복잡하거나 무거운 연산 (localStorage / sessionStorage 접근, map, filter, find 배열 접근)

### 3.1.2 useEffect

- 클래스형 컴포넌트 생명주기 메서드와 비슷한 작동 구현
- 의존성에 있는 값이 이전과 다르면 부수효과 실행
- **클린업 함수의 목적** : 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지, 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는 청소해주는 개념
- 의존성 배열 : 빈배열은 최초 렌더링 시 실행 후 실행되지 않음
    
    ```jsx
    // 1
    function Component() {
    	console.log('렌더링됨')
    }
    
    // 2
    function Component() {
    	useEffect(() => {
    		console.log('렌더링됨')
    	})
    }
    ```
    
    - 서버사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해줌
        - window 객체의 접근에 의존하는 코드 사용가능
    - useEffect는 컴포넌트 렌더링의 부수효과, 즉 컴포넌트의 렌더링이 완료된 후 실행됨. 반면 직접실행은 렌더링 도중 실행됨. 따라서 1번과 달리 서버 사이드 렌더링의 경우 서버에서도 실행됨. 이는 함수형 컴포넌트의 반환을 지연 - 성능의 악영향
- useEffect 구현

<aside>
💡 useEffect 구현과정 (pg. 203)

1. 이전 훅 정보 확인
2. 변경되었는지 확인. 
    1. 이전값이 있다면 이전값을 얕은 비교로 비교해 변경이 일어났는지 확인
    2. 없다면 최초 실행이므로 변경이 일어난 것으로 간주 실행 유도
3. 변경이 일어났다면 첫 번째 인수인 콜백 함수 실행
4. 현재 의존성을 훅에 다시 저장
5. 다음 훅이 일어날 때를 대비하기 위해 index 추가
</aside>

- 사용시 주의할 점
    - **eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제** : useEffect 내부에서 사용하는 값 중 의존성 배열에 포함되어 있지 않은 값이 있을 때 경고 (버그 확률 큼)
    - **거대한 useEffect 만들지 마라** : 가볍게 유지, useCallback,useMemo로 정제한 내용만 useEffect 담기
    - **불필요한 외부 함수 만들지 말기** : 내부에서 정의해서 사용하는 편이 도움됨

### 3.1.3 useMemo

- 비용이 큰 연산에 대한 결과를 저장해두고, 이를 반환하는 훅 (최적화)
- 첫번째 인수로는 값을 반환하는 생성 함수, 두번째 인수로는 해당 함수가 의존하는 값의 배열 전달
- 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으며 함수를 재실행 하지 않고 이전에 기억해 둔 값을 반환, 변했다면 함수 실행 후 값 반환(기억) , 컴포넌트도 가능하지만 React.Memo쓰는편이 낫다.

### 3.1.4 useCallback

- useMemo는 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백 자체를 기억한다. 특정함수 재사용

### 3.1.5 useRef

- 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경가능함
- 값이 변하더라도 렌더링 발생시키지 않음
- useMemo로 useRef 구현 가능함

### 3.1.6 useContext

- Context란? : 컴포넌트의 거리가 멀어질수록 props drilling 때문에 불편해짐, 이를 context이용하면 하위 컴포넌트에서 갑사 사용 가능
- **Context를 함수형 컴포넌트에서 사용할 수 있게 해주는 useContext 훅**
    - 상위컴포넌트 어딘가에서 선언된 <Context.Provider /> 에서 제공한 값을 사용할 수 있음
    - 여러개의 provider가 있다면 가장 가까운 값을 가져옴
    - 컴포넌트 트리가 복잡해질 수록 사용 어려워짐, 이를 방지하기 위해 내부에서 해당 콘텍스트가 존재하는 환경인지, 한번이라도 초기화되어 값을 내려주고 있는지 확인해보면 됨
- 사용시 주의할 점
    - 컴포넌트 내부에서 사용시 컴포넌트의 재사용이 어려워짐
    - 이를 사용하는 컴포넌트를 최대한 작은 단위로 만들거나 재사용 하지 않을만한 컴포넌트에서 사용
    - 상태관리를 위한 API가 아닌 상태를 주입해주는 API 아래조건 만족
        - 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 함
        - 필요에 따라 이러한 상태 변화를 최적화 할 수 있어야 함

### 3.1.7 useReducer

- useState 심화버전, 복잡한 상태값을 미리 정의해놓은 시나리오로 관리 가능
- state : 첫번째 요소
- dispatcher : state를 업데이트하는 함수
- reducer : action을 정의하는 함수
- initialState : 초기값 설정
- init : 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수

### 3.1.8 useImperativeHandle

- **forwardRef** : ref 받고자 하는 컴포넌트를 forwardRef로 감싸고, 두번째 인수로 ref를 전달받음. 부모컴포넌트에서는 동일하게 props.ref 통해 ref 넘겨주면 됨
    - ref를 props로 전달하고 전달받은 컴포넌트에서도 ref 라는 이름 그대로 사용 가능
- useImperativeHandle :  부모에게서 넘겨받은 ref를 원하는대로 수정할 수 있는 훅

### 3.1.9 useLayoutEffect

- useEffect와 동일하나 모든 DOM의 변경 후에 동기적으로 발생

### 3.1.10 useDebugValue

- 디버깅하고 싶은 정보를 훅에다가 사용하면 개발자 도구에서 사용 가능

### 3.1.11 훅의 규칙

- 최상위에서만 훅 호출. 반복문, 조건문 중첩된 함수 내에서 훅 실행 불가. 이순서를 따라야 렌더링될때마다 동일한 순서로 훅이 호출됨
- 함수형 컴포넌트, 사용자 정의 훅 두가지 경우만 호출 가능

### 3.1.12 정리

제대로 작동방식을 이해했을때 더 짜임새 있는 개발이 가능하다.

---

## 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

중복코드는 피해야 한다 다 알고 있을 것이다. 두가지가 어떻게 쓰이는지 공통된 코드를 하나로 만들고자 할 때 어떤 것을 선택해야 하는지 알아보자

### 3.2.1 사용자 정의 훅

- 서로 다른 컴포넌트 내부에서 같은 로직을 공유할 때

### 3.2.2 고차 컴포넌트

- 컴포넌트 자체의 로직을 재사용. 고차함수의 일종으로 리액트가 아닌 환경에서도 사용 가능
- **React.memo**
- **고차 함수 만들어보기** - pg.246
- **고차 함수를 활용한 리액트 고차 컴포넌트 만들어보기** - pg.247

### 3.2.3 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

- **사용자 정의 훅이 필요한 경우**
    - useEffect, useState와 같이 훅으로만 공통로직을 격리할 수 있을때
    - 컴포넌트 내부에 미치는 영향을 최소화해 개발자가 원하는 방향으로만 사용가능
    - 부수효과가 비교적 제한적
    - 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶을 때
- **고차컴포넌트 사용해야 하는경우**
    - 반환값. 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용
    - 공통화된 렌더링 로직을 처리하기 적합하지만 고차 컴포넌트가 많아질수록 복잡성이 증가하므로 유의

### 3.2.4 정리

적절한 방법을 상황에 맞게 사용하자


---

# 5장 리액트와 상태 관리 라이브러리

## 5.1 상태 관리는 왜 필요한가?

**웹 애플리케이션의 상태**

- UI
- URL
- FORM
- 서버에서 가져온 값

### 5.1.1 리액트 상태 관리의 역사

- **Flux 패턴의 등장**
    - 전역상태관리 : Context API는 상태주입… 리덕스 플럭스 이전의 상태..
    - 많은양의 데이터가 어디서 변화했는지 알지 못함 (양방향 바인딩 문제)
    - 단방향 바인딩 Flux 제안
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/bd55ef60-767c-46fd-af1e-7c912e2b2167/886473cb-389c-40c7-96f7-b714356296e3/Untitled.png)
        
        - action : 어떠한 방법을 처리할 액션과 함께 포함시킬 데이터를 의미
        - dispatcher : 액션을 스토어로 보냄. 콜백함수형태로 액션이 정의한 타입과 데이터를 모두 보냄
        - store : 실제 상태에 따른 값과 상태 변경을 위한 메서드 정의
        - view : 컴포넌트. 스토어에서 만들어진 데이터를 가져와 화면에 렌더링
- **시장 지배자 Redux 등장**
    - Flux에 Elm(웹페이지를 선언적으로 작성하기 위한 언어)아키텍쳐 도입
        - model : 애플리케이션의 상태
        - view : 모델을 표현하는 HTML
        - update : 모델을 수정하는 방식
    - 스토어에 저장된 상태 객체를 디스패치를 통해 업데이트하며 이러한 작업은 리듀서 함수를 통해 새로운 복사본을 반환한 다음 이를 전파함
    - 할일이 너무 많음 하지만 지금은 많이 간소화됨
- **Context API와 useContext**
    - props 넘겨주지 않더라도 Context API 사용해서 Provider 주입가능한 새로운 Context API 등장
    - 상태를 주입해주는 개념이기에 주의가 필요함
- **훅의 탄생, 그리고 React Query와 SWR**
    - 외부에서 데이터를 불러오는 fetch(HTTP 요청)를 관리하는데 특화된 라이브러리
- **Recoil, Zustand, Jotai, Valtio에 이르기까지**
    - 범용적으로 쓸 수 있는 상태 관리 라이브러리

### 5.1.2 정리

상태관리 라이브러리가 건강한 커뮤니티 속에서 발전하고 있으니 올바른 선택기준을 찾는게 중요하다

---

## 5.2 리액트 훅으로 시작하는 상태 관리

리액트에서 가장 중요한 상태관리

### 5.2.1 가장 기본적인 방법 : useState와 useReducer

- 지역적 상태관리 방법. 컴포넌트 재 렌더링으로 인한 초기화를 피하고 싶다면 부모 컴포넌트에서 상속 받을 것

### 5.2.2 지역 상태의 한계를 벗어나보자 : useState의 상태를 바깥으로 분리하기

- 리액트 렌더링 원리에 의해 컴포넌트 외부로 값을 빼더라도 **리렌더** 트리거를 넣어줘야함
- 인터페이스와 사용자의 관점에서 subscribe 사용 하고, subscribe에 상태를 변경하는 함수를 등록

### 5.2.3 useState와 Context를 동시에 사용해 보기

- 여러개의 서로 다른 데이터를 공유하고 싶을 때 사용
- Context를 활용해 해당 스토어를 하위 컴포넌트에 주입

### 5.2.4 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

- **Recoil**
    - RecoilRoot 선언, 하나의 스토어 생성
    - atom이라는 상태 단위를 스토어에 등록
    - atom은 key 값으로 구별
    - hook을 통해 atom의 변화를 구독 즉 파생값 만들 수 있음
    - 값이 변경된다면 forceUpdate 같은 기법을 통해 히렌더링, 최신 stom 값 가져옴
- Jotai
    - Recoil 컨셉을 가져와 사용하고 단점을 해결하여 사용
    - Bottom-up 접근법을 취하고 있어 불필요한 렌더링이 일어나지 않음
    - Key를 별도로 관리하는 부분을 추상화 하여 사용자가 관리할 필요가 없음
    - 객체 참조를 WeakMap에 보관해 해당 객체 자체가 변경 되지 않는 한 객체의 참조를 통해 관리 가능
- Zustand
    - 리덕스에 영감받아 만들어짐. 중앙 집중형
    - 빠르고 간단하게 스토어를 만들어 사용
    - 특정 프레임워크에 종속적이지 않음
    - 번들사이즈 작음