### 📅 2024년 5월 1일

# 📚 12장 모든 웹 개발자가 관심을 가져야 할 핵심 웹 지표

## 핵심 웹 지표
- 구글에서 만든 지표
- 웹사이트에서 뛰어난 사용자 경험을 제공하는 데 필수적인 지표

### LCP(Largest Contentful Paint) 
- 페이지가 처음으로 로드를 시작한 시점부터 뷰포트 내부에서 가장 큰 이미지 또는 텍스트를 렌더링하는 데 걸리는 시간.
- 즉, 사용자의 기기가 노출하는 뷰포트 내부에서 가장 큰 영역을 차지하는 요소가 렌더링되는 데 얼마나 걸리는지 측정하는 지표.
#### 뷰포트
- 사용자에게 현재 노출되는 화면
- 뷰포트는 기기마다 다르다.

#### 뷰포트 내에서 '큰 이미지와 텍스트'
- `<img/>`
- `<svg/>` 내부 `<image/>`
- `<video/>`
- url()로 불러온 배경 이미지가 있는 요소
- 텍스트와 같이 인라인 텍스트 요소를 포함하고 있는 블록레벨 요소


#### 의미
- 사용자가 페이지 로딩을 체감하기 위해서는 꼭 모든 페이지가 완전히 로딩될 필요는 없음.
- 사용자에게 노출되는 부분, 즉 뷰포트내에 위치하는 컨텐츠만 로딩되어 있다면, 사용자는 페이지가 로딩이 완료되었다고 체감할 것.
- 따라서, 사용자에게 페이지의 정보를 화면에 전달하는 속도를 객관적으로 판단하기 위한 지표로 만들어진 것이 LCP

#### 개선방안
- LCP 예상영역에 문자열을 넣자 => 이미지 넣어달라고 하면 어떡해,,,
- 그래도 이미지를 넣고싶어요...

```html
<!-- 1. -->
<img />
<!-- 2. -->
<svg>
    <image/>
</svg>
<!-- 3. -->
<video/>
```

- 1번과 3번의 예제코드가 가장 빠르게 완성됨.


1. img: 브라우저의 프리로드 스캐너에 의해 먼저 발견되어 빠르게 요청이 일어남.
   - 프리로드 스캐너란, HTML을 파싱하는 단계를 차단하지 않고 이미지와 같이 빠르게 미리 로딩하면 좋은 리소스를 찾아 로딩하는 브라우저의 기능.
2. svg: img가 미처 로딩되지 않고, svg만 로딩된 시점에 이미 LCP 가 완료된 것으로 간주함. 그리고, 모든 리소스를 다 불러온 이후에 이미지를 불러옴.
3. 프리로드 스캐너에 의해 조기에 발견됨.
4. background-image, url: CSS에 있는 리소스는 항상 느림. 왜냐면 브라우저가 해당 리소스를 필요로 하는 DOM을 그릴 준비가 될 때 까지 리소스 요청을 뒤로 미루기 때문.

### FID (First Input Play)
- 최초 입력 지연
- 사용자가 페이지와 처음 상호 작용할 때 부터 해당 상호 작용에 대한 응답으로 브라우저가 실제로 이벤트 핸들러 처리를 시작하기까지의 시간을 측정.
- 사용자가 얼마나 빠르게 웹페이지아의 상호작용에 대한 응답을 받을 수 있는지 측정하는 지표.
- 최초의 입력 하나에 대해서만 그 응답지연이 얼마나 걸리는지 판단.

#### 의미
- 웹사이트 내부 이벤트가 반응이 늦어지는 이유
  - 메인 스레드가 바쁘기 때문
  - 왜 바쁠까?
    - 대규모 렌더링이 일어나거나, 다른 작업을 처리하는 데 리소스를 할애하고 있기 때문.
    - 자바스크립트는 싱글 스레드이기 때문에 자바스크립트가 다른 작업을 실행할 수 없어 지연이 일어남.
- 화면이 최초에 그려지고 난 뒤, 사용자가 웹페이지에 클릭 등 상호작용을 수행했을 때 메인 스레드가 이 이벤트에 대한 반응을 할 수 있을 때까지 걸리는 시간을 의미.

#### 개선 방안
1. 실행에 오래 걸리는 긴 작업을 분리.
 - React의 Suspense 와 lazy
 - Next.js 의 dynamic

2. 자바스크립트 코드의 최소화
   - 번들러가 알아서 처리해주지만..
   - 개발자도구 - 커버리지 도구를 통해 웹페이지에서 사용되지 않은 코드를 확인할 수 있음.
   - 폴리필(브라우저에서 지원하지 않는 기능을 사용하기 위해 직접 구현하고 집어넣는 코드) 확인

3. 타사 자바스크립트 코드 실행의 지연
   - defer: 해당 스크립트를 다른 리소스와 함께 병렬로 다운로드. 다운로드가 완료되었다고 하더라도 페이지가 완전히 로딩된 이후 맨 마지막에 실행
   - async: 다른 리소스와 함께 병렬로 다운로드. 다운로드가 완료된 순서대로 실행
   - 둘다 없으면: script를 만나는 순간 다운로드가 우선. 

### CLS(Cumulative Layout Shift)
- 누적 레이아웃 이동

#### 의미
- 사용자의 가시적인 콘텐츠에 영향을 미쳐야 하기 때문에 뷰포트 내부의 요소에 대해서만 측정함.
- 최초 렌더링이 시작된 위치에서 만약 레이아웃의 이동이 발생한다면 누적 레이아웃 이동 점수로 기록함.

#### 개선 방안
1. 삽입이 예상되는 요소를 위한 추가적인 공간 확보
   - 스켈레톤 UI
2. 폰트 로딩 최적화
   - FOUT
   - FOIT
    - <link/>의 preload 사용
3. 적절한 이미지 크기 설정
    

### TTFB(Time To First Byte)
- 브라우저가 웹페이지의 첫 번째 바이트를 수신하는 데 걸리는 시간
- 최초의 응답이 오는 바이트가 얼마나 걸리는지 측정하는 지표

### FCP(First Contentful Paint)
- 페이지가 로드되기 시작한 시점부터 페이지 콘텐츠 일부가 화면에 렌더링될 때 까지의 시간


# 📚 13장 웹페이지의 성능을 측정하는 다양한 방법
## 구글 라이트하우스
- 성능
  - FCP, LCP, CLS
- 접근성
  - 웹 접근성
- 권장사항
  - 웹사이트 개발시 고려해야할 요소들을 얼마나 지키고 있는지
- 검색엔진최적화

