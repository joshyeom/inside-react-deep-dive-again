## 컴포넌트

컴포넌트는 추상화 된 코드조각으로, 재사용 가능하고 서로 다른 컴포넌트를 사용할 수 있습니다.

훌륭한 컴포넌트는 명확한 API(props)를 가지고, 내부의 복잡한 구현은 숨기며(캡슐화), 예측 가능하게 동작해야 합니다.

컴포넌트를 미리 만들어두면, 다른 프로젝트에서 화면 구현 소요 시간을 줄 일 수 있습니다. 그에 따라 추가 요구사항과 변경 사항에 대응하기 쉽고, 이는 팀 전체의 생산성을 향상시켜줍니다.

그러므로 프론트엔드 개발자는 단순히 CSS를 이용해 화면에 UI를 그리는 걸 넘어서 본인이 작성하는 UI 컴포넌트가 제품에 전반적인 영향을 미친다는 사실을 인지하고 있어야 합니다.

## 컴포넌트와 프레임워크의 관계

리액트를 처음 배운 사람으로써는, 컴포넌트는 리액트와 같은 프레임워크의 전유물처럼 느껴집니다. 하지만, 컴포넌트는 프레임워크만의 전유물이 아니라 단지 프레임워크로 컴포넌트를 쉽게 만들 수 있는 도구 정도입니다.

순수한 자바스크립트만으로 어떻게 UI를 독립적인 조각으로 나눌 수 있을까? 와 같은 고민은 특정 기술에 종속되지 않는 유연한 개발자로 만들어주는 강력한 무기가 될 수 있으므로, 프레임워크 없이 컴포넌트를 설계하고 만들 줄 알아야 합니다.

프레임 워크의 편리함에만 의존하면 '어떻게' 만들지 집중할 수 있지만, 컴포넌트의 위상을 생각해보면 '무엇을' 만들고 '어떻게' 상호작용해야 하는지에 대한 설계 관점으로 시야가 확장될 수 있습니다.

## 웹 컴포넌트 API

웹 컴포넌트라는 표준기술의 등장으로 UI 컴포넌트라는 개념이 더 이상 특정 프레임워크의 전유물이 아니게 되었습니다.

## 추상화와 인터페이스

공통 특성을 일반화, 복잡한 세부사항을 감춰 간결한 모델로 표현하는 것이 추상화 작업입니다.

추상화된 컴포넌트가 외부에 약속 (contract)으로 제시하는 공개 API의 집합을 인터페이스라고 합니다.

## 바닐라 자바스크립트로 컴포넌트 구현하기

```html
...
<body>
    <div class="network-indicator">
        Network Speed: <span id="networkSpeed">Fast 3G</span>
    </div>

    <div class="controls">
        <h3>Simulation Controls</h3>
        <div>
            <label>Network Speed:</label>
            <select id="networkSelect">
                <option value="fast3g">Fast 3G</option>
                <option value="slow3g">Slow 3G</option>
                <option value="2g">2G</option>
            </select>
        </div>
        <div style="margin-top: 10px;">
            <button onclick="resetSimulation()">Reset Images</button>
        </div>
    </div>
    <div id="imageContainer"></div>

    <script>
        const NETWORK_CONDITIONS = {    // 각 네트워크 상황에 따른 지연
            fast3g: { min: 500, max: 1500 },
            slow3g: { min: 1000, max: 3000 },
            '2g': { min: 2000, max: 5000 }
        };

        class LazyImageLoader {
            #loadingStates; // private 필드로 loading 상태를 캡슐화

            constructor(options = {}) {
                this.options = {    // 옵션으로 받은 데이터로 effect를 설정
                    root: null,
                    rootMargin: '50px 0px',
                    threshold: 0.1,
                    blurEffect: options.blurEffect || false,
                    loadingIndicator: options.loadingIndicator || false,
                    blurAmount: options.blurAmount || '100px',
                    loadingSpinnerColor: options.loadingSpinnerColor || 'green',
                    ...options
                };

                this.observer = new IntersectionObserver( // 인스턴스에 옵저버 생성
                    this.#handleIntersection.bind(this),
                    {
                        root: this.options.root,
                        rootMargin: this.options.rootMargin,
                        threshold: this.options.threshold
                    }
                );

                this.#loadingStates = new WeakMap();    // WeakMap을 사용하여 이미지 객체가 DOM에서 제거되면 자동으로 가비지 컬렉터에서 제거
            }

            observe(selector) { // 이미지들을 선택하고 각 이미지에 lazy loading 설정
                const images = document.querySelectorAll(selector);
                images.forEach(img => this.#setupImage(img));
            }

            #setupImage(img) {  // 이미지에 lazy loading을 설정하는 함수
                if (img.dataset.lazyLoaderInitialized) return;

                img.dataset.lazyLoaderInitialized = 'true'; // 중복방지용 이미지 초기화
                const originalSrc = img.src;
                img.dataset.src = originalSrc;

                const wrapper = document.createElement('div'); // 필터 wrapper 생성 및 스타일링
                wrapper.style.position = 'relative';
                wrapper.style.overflow = 'hidden';
                img.parentNode.insertBefore(wrapper, img);  // 이미지의 부모 요소에 wrapper 요소를 이미지보다 먼저 삽입
                wrapper.appendChild(img); // wrapper안에 이미지

                img.style.transition = 'filter 0.3s ease-out';

                if (this.options.blurEffect) {
                    img.style.filter = `blur(${this.options.blurAmount})`;
                }

                if (this.options.loadingIndicator) {
                    const spinner = this.#createLoadingSpinner();
                    wrapper.appendChild(spinner);
                    this.#loadingStates.set(img, { spinner });
                }

                img.src = ''; // 이미지 src 비우기
                this.observer.observe(img); // 옵저버 등록 => 화면에 보이면 이미지 로딩 시작
            }

            #handleIntersection(entries, observer) { // 옵저버 콜백: 이미지가 화면에 보이는지 확인
                entries.forEach(entry => {
                    if (entry.isIntersecting) { // 이미지가 화면에 보이면
                        this.#loadImage(entry.target); // 이미지 로딩
                        observer.unobserve(entry.target); // 옵저버 해제
                    }
                });
            }

            #loadImage(img) {
                const originalSrc = img.dataset.src;

                const networkSpeed = document.getElementById('networkSelect').value;
                const { min, max } = NETWORK_CONDITIONS[networkSpeed];
                const delay = Math.random() * (max - min) + min;

                setTimeout(() => { // 네트워크 지연 시뮬레이션 후 이미지 로드 및 효과 제거
                    img.src = originalSrc;

                    if (this.options.blurEffect) {
                        img.style.filter = 'blur(0)';
                    }

                    if (this.options.loadingIndicator) {
                        const state = this.#loadingStates.get(img);
                        if (state && state.spinner) {
                            state.spinner.remove();
                        }
                    }
                }, delay);
            }

            #createLoadingSpinner() {
                const spinner = document.createElement('div');
                spinner.style.cssText = `
                    position: absolute;
                    top: 50%;
                    left: 50%;
                    transform: translate(-50%, -50%);
                    width: 30px;
                    height: 30px;
                    border: 3px solid #f3f3f3;
                    border-top: 3px solid ${this.options.loadingSpinnerColor};
                    border-radius: 50%;
                    animation: spin 1s linear infinite;
                `;

                if (!document.querySelector('#lazy-loader-spinner-keyframes')) {
                    const style = document.createElement('style');
                    style.id = 'lazy-loader-spinner-keyframes';
                    style.textContent = `
                        @keyframes spin {
                            0% { transform: translate(-50%, -50%) rotate(0deg); }
                            100% { transform: translate(-50%, -50%) rotate(360deg); }
                        }
                    `;
                    document.head.appendChild(style);
                }

                return spinner;
            }
        }

        const loader = new LazyImageLoader({  // 로더 선언
            blurEffect: true,
            loadingIndicator: true,
            blurAmount: '10px',
            loadingSpinnerColor: 'green'
        });

        function generateImages(count = 10) {
            const container = document.getElementById('imageContainer');
            container.innerHTML = ''; // 기존 이미지 모두 제거 (초기화)

            for (let i = 0; i < count; i++) {   // 이미지 카운트 만큼 생성
                const wrapper = document.createElement('div');
                wrapper.className = 'image-container';

                const img = document.createElement('img');
                img.className = 'lazy-image';
                img.src = `https://picsum.photos/800/400`;
                img.alt = `Lazy loaded image ${i + 1}`;

                wrapper.appendChild(img);
                container.appendChild(wrapper);
            }

            loader.observe('.lazy-image'); // 모든 lazy-image 등록
        }

        document.getElementById('networkSelect').addEventListener('change', function() {
            document.getElementById('networkSpeed').textContent =
                this.options[this.selectedIndex].text;
        });

        function resetSimulation() {
            generateImages();
        }

        generateImages();
    </script>
</body>
</html>
```

이 코드에서 LazyImageLoader만 따로 모듈로 변경하여 컴포넌트로 사용할 수 있습니다.

```js
export class LazyImageLoader {
  #loadingStates;

  constructor(options = {}) {
    this.options = {
      root: null,
      rootMargin: "50px 0px",
      threshold: 0.1,
      blurEffect: options.blurEffect || false,
      loadingIndicator: options.loadingIndicator || false,
      blurAmount: options.blurAmount || "5px",
      loadingSpinnerColor: options.loadingSpinnerColor || "green",
      ...options,
    };

    this.observer = new IntersectionObserver(
      this.#handleIntersection.bind(this),
      {
        root: this.options.root,
        rootMargin: this.options.rootMargin,
        threshold: this.options.threshold,
      }
    );

    this.#loadingStates = new WeakMap();
  }

  observe(selector) {
    const images = document.querySelectorAll(selector);
    images.forEach((img) => this.#setupImage(img));
  }

  #setupImage(img) {
    if (img.dataset.lazyLoaderInitialized) return;

    img.dataset.lazyLoaderInitialized = "true";
    const originalSrc = img.src;
    img.dataset.src = originalSrc;

    const wrapper = document.createElement("div");
    wrapper.style.position = "relative";
    wrapper.style.overflow = "hidden";
    img.parentNode.insertBefore(wrapper, img);
    wrapper.appendChild(img);

    img.style.transition = "filter 0.3s ease-out";

    if (this.options.blurEffect) {
      img.style.filter = `blur(${this.options.blurAmount})`;
    }

    if (this.options.loadingIndicator) {
      const spinner = this.#createLoadingSpinner();
      wrapper.appendChild(spinner);
      this.#loadingStates.set(img, { spinner });
    }

    img.src = "";
    this.observer.observe(img);
  }

  #handleIntersection(entries, observer) {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        this.#loadImage(entry.target);
        observer.unobserve(entry.target);
      }
    });
  }

  #loadImage(img) {
    const originalSrc = img.dataset.src;

    // Simulate network delay
    const networkSpeed = document.getElementById("networkSelect").value;
    const { min, max } = NETWORK_CONDITIONS[networkSpeed];
    const delay = Math.random() * (max - min) + min;

    setTimeout(() => {
      img.src = originalSrc;

      if (this.options.blurEffect) {
        img.style.filter = "blur(0)";
      }

      if (this.options.loadingIndicator) {
        const state = this.#loadingStates.get(img);
        if (state && state.spinner) {
          state.spinner.remove();
        }
      }
    }, delay);
  }

  #createLoadingSpinner() {
    const spinner = document.createElement("div");
    spinner.style.cssText = `
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 30px;
            height: 30px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid ${this.options.loadingSpinnerColor};
            border-radius: 50%;
            animation: spin 1s linear infinite;
        `;

    if (!document.querySelector("#lazy-loader-spinner-keyframes")) {
      const style = document.createElement("style");
      style.id = "lazy-loader-spinner-keyframes";
      style.textContent = `
                @keyframes spin {
                    0% { transform: translate(-50%, -50%) rotate(0deg); }
                    100% { transform: translate(-50%, -50%) rotate(360deg); }
                }
            `;
      document.head.appendChild(style);
    }

    return spinner;
  }
}
```

### 웹 컴포넌트로 컴포넌트 구현하기

```html
...
<body>
    <div class="network-indicator">
        Network Speed: <span id="networkSpeed">Fast 3G</span>
    </div>
    <div class="controls">
        <h3>Simulation Controls</h3>
        <div>
            <label>Network Speed:</label>
            <select id="networkSelect">
                <option value="fast3g">Fast 3G</option>
                <option value="slow3g">Slow 3G</option>
                <option value="2g">2G</option>
            </select>
        </div>
        <div style="margin-top: 10px;">
            <button onclick="resetSimulation()">Reset Images</button>
        </div>
    </div>

    <!-- 커스텀 태그로 사용 가능 -->
    <lazy-image data-src="https://picsum.photos/800/400" alt="Placeholder Image 1" data-network-speed="fast3g" indicator-color="green" border-radius="10px"></lazy-image>
    <lazy-image data-src="https://picsum.photos/800/400" alt="Placeholder Image 2" data-network-speed="fast3g" indicator-color="blue" apply-blur border-radius="10px"></lazy-image>
    <lazy-image data-src="https://picsum.photos/800/400" alt="Placeholder Image 3" data-network-speed="fast3g" indicator-color="red" border-radius="10px"></lazy-image>
    <lazy-image data-src="https://picsum.photos/800/400" alt="Placeholder Image 4" data-network-speed="fast3g" indicator-color="purple" apply-blur border-radius="10px"></lazy-image>
    <lazy-image data-src="https://picsum.photos/800/400" alt="Placeholder Image 5" data-network-speed="fast3g" indicator-color="blue" apply-blur border-radius="10px"></lazy-image>
    <script src="./LazyImageLoader.js"></script>
    <script>
        // lazy-image 태그를 LazyImageLoader 클래스로 정의
        customElements.define('lazy-image', LazyImageLoader);
        // 초기화 함수
        function resetSimulation() {
            const lazyImages = document.querySelectorAll('lazy-image');
            const networkSpeed = document.getElementById('networkSelect').value;
            lazyImages.forEach(img => {
                img.setAttribute('data-network-speed', networkSpeed);
                img.reset();
                img.getObserver().observe(img);
            });
        }
    </script>
</body>
</html>
```

HTML에서 손쉽게 컴포넌트로 사용 가능합니다.

```js
class LazyImageLoader extends HTMLElement {
  constructor() {
    // super로 HTMLElement의 constructor()를 실행
    super();
    this._image = document.createElement("img");
    this._image.style.opacity = "0";
    this._image.style.transition = "opacity 1s ease-in-out";
    this._spinner = document.createElement("div");
    this._spinner.className = "loader";
    this._spinner.style.display = "block";

    // 인스턴스 요소에서 속성 값 가져오기
    const indicatorColor = this.getAttribute("indicator-color");
    if (indicatorColor) {
      this._spinner.style.borderTopColor = indicatorColor;
    }
    this.applyBlur = this.hasAttribute("apply-blur");
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      {
        rootMargin: "0px 0px 200px 0px",
        threshold: 0,
      }
    );
  }

  // React의 props처럼 속성 감시
  static get observedAttributes() {
    return ["border-radius"];
  }

  // border-radius 변경 함수
  attributeChangedCallback(name, oldValue, newValue) {
    if (name === "border-radius") {
      this._image.style.borderRadius = newValue;
    }
  }
  getObserver() {
    return this.observer;
  }
  connectedCallback() {
    // 쉐도우 DOM에 연결
    const shadowRoot = this.attachShadow({ mode: "closed" });
    const style = document.createElement("style");
    style.textContent = `
      .loader {
        border: 16px solid #f3f3f3;
        border-top: 16px solid blue;
        border-radius: 50%;
        width: 50px;
        height: 50px;
        animation: spin 2s linear infinite;
      }
      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
    `;

    //쉐도우 돔에 주입
    shadowRoot.appendChild(style);
    shadowRoot.appendChild(this._spinner);
    shadowRoot.appendChild(this._image);
    this.observer.observe(this);
  }
  disconnectedCallback() {
    this.observer.unobserve(this);
  }
  handleIntersection(entries) {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        this.loadImage();
        this.observer.unobserve(this);
      }
    });
  }
  loadImage() {
    const networkSpeed = this.getAttribute("data-network-speed");
    let delay = 0;
    if (networkSpeed === "slow3g") {
      delay = 2000;
    } else if (networkSpeed === "2g") {
      delay = 5000;
    }
    if (this.applyBlur) {
      this.style.filter = "blur(10px)";
    }
    setTimeout(() => {
      this._image.src = this.getAttribute("data-src");
      this._image.onload = () => {
        this._image.style.opacity = "1";
        this.style.filter = "blur(0)";
        this._spinner.style.display = "none";
      };
    }, delay);
  }
  reset() {
    this._image.src = "";
    this._image.style.opacity = "0";
    this._image.style.display = "block";
    this._spinner.style.display = "block";
  }
}
```

### 쉐도우 돔

컴포넌트의 스타일과 구조를 독립적으로 유지하여 외부의 영향을 받지 않는 DOM입니다.

```js
connectedCallback() {
    // 쉐도우 DOM에 연결
    const shadowRoot = this.attachShadow({ mode: "closed" });
    const style = document.createElement("style");
    style.textContent = `
      .loader {
        border: 16px solid #f3f3f3;
        border-top: 16px solid blue;
        border-radius: 50%;
        width: 50px;
        height: 50px;
        animation: spin 2s linear infinite;
      }
      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
    `;

    //쉐도우 돔에 주입
    shadowRoot.appendChild(style);
    shadowRoot.appendChild(this._spinner);
    shadowRoot.appendChild(this._image);
    this.observer.observe(this);
  }
```

`attachShadow` 메서드로 쉐도우 DOM과 객체를 연결하고, 스타일을 새로 정의 후 쉐도우 DOM에 넣어줍니다. 이렇게되면 쉐도우 DOM은 이 요소만을 위한 스타일을 렌더링 합니다. 이 스타일은 외부가 격리되어있어 서로 영향을 주고받지 않습니다. 이로써, 격리된 하나의 컴포넌트처럼 동작하게 할 수 있습니다.

아쉬운점은 전역 스타일이 적용이 안된다는 점입니다. 또한, 이벤트 위임이 복잡해지는 문제가 있습니다.

```js
document.addEventListener("click", (e) => {
  console.log(e.target); // Shadow DOM 내부 요소 아님
  console.log(e.composedPath()); // 이걸로 확인해야 함
});
```

## 아하 모먼트

리액트로 프론트엔드를 입문하는 바람에 '컴포넌트'는 JSX로 만들어진 외부 모듈정도로 생각했었습니다. 하지만, 컴포넌트는 더 확장된 개념이고, 확장된 개념으로 바라보아야 '어떻게' 구현할지보다 '무엇을' 만들고 '어떻게' 상호작용할지를 고민할 필요가 있다고 깨달았습니다.

또한, 깊게 고민하여 만들어낸 컴포넌트는 팀의 전체적인 생산성을 올릴 수 있지만 반대로 별 고민없이 만들어낸 컴포넌트는 팀의 퍼포먼스를 낮출 수 있다는 생각이 들었습니다.

바닐라 자바스크립트 컴포넌트를 직접 생성해보면서 React의 Class 컴포넌트가 생각났습니다. 오랜만에 Class를 사용하여 this 바인딩이 어려웠는데 그림을 그려가면서 바인딩을 익히니 다시 이해가 되었습니다.

```js
this.observer = new IntersectionObserver(
  this.#handleIntersection.bind(this), // this에서 프로퍼티를 생성한뒤에 왜 다시 this에 바인딩하지? 라는 의문이 있었습니다.
  {
    root: this.options.root,
    rootMargin: this.options.rootMargin,
    threshold: this.options.threshold,
  }
);

// IntersectionObserver의 콜백이 나중에 실행 될때 콜백의 this가 인스턴스를 가리키ㅣㅈ 않으므로 미리 인스턴스 this에 고정을 시켜놓습니다.
```

웹 컴포넌트 API로 컴포넌트를 직접 생성해보면서 컴포넌트가 React만의 전유물이 아님을 알 수 있었고, 쉐도우 DOM이라는 개념을 새로 익힐 수 있었습니다.쉐도우 DOM을 사용하면 스타일과 DOM을 격리하여 독립적으로 사용할 수 있지만, 전역 스타일 적용 불가와 이벤트 위임 복잡도 증가라는 트레이드오프가 있습니다."
