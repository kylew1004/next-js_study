# Create Project

```
Next.js 프로젝트 초기화
npx create-next-app@latest

Next.js 프로젝트 초기화(+타입스크립트)
npx create-next-app@latest --typescript
```

# library 와 framework의 차이

- 라이브러리와 프레임워크의 주요 차이점은 "Inversion of Control"(통제의 역전)입니다.
  라이브러리에서 메서드를 호출하면 사용자가 제어할 수 있습니다.
  그러나 프레임워크에서는 제어가 역전되어 프레임워크가 사용자를 호출합니다.

- 라이브러리
  - 사용자가 파일 이름이나 구조 등을 정하고, 모든 결정을 내림
- 프레임워크
  - 파일 이름이나 구조 등을 정해진 규칙에 따라 만들고 따름

# NextJS 앱과 create-react-app 앱간의 차이?

- create-react-app에서는 모든 것이 클라이언트-사이드-렌더링(SCR)입니다. (때때로 유저에게 하얀창을 보여주는거)
  브라우저가 react의 코드를 받고 모든 걸 앞에 그려주기 때문입니다.

- NextJS에서는 모든 페이지가 미리 렌더링 됩니다. 그건 HTML페이지가 됩니다.
  자바스크립트 요소들이 하나도 없는 상태가 되는데 특정 JS 모듈 뿐 아니라 단순 클릭과 같은 이벤트 리스너들이 각 웹 페이지의 DOM 요소에 하나도 적용되어 있지 않은 상태입니다. Next.js Server에서는 Pre-Rendering된 웹 페이지를 클라이언트에게 보내고 나서, 바로 리액트가 번들링 된 자바스크립트 코드들을 클라이언트에게 전송합니다.
  이를 'Hydrate'이라고 합니다.

# Link 태그

- <a href>를 쓰면 페이지가 새로고침이 되는데 side render를 하는 의미가 없어지므로 Link태그를 활용합니다.
- 페이지 간 클라이언트 측 경로 전환을 활성화하고 single-page app 경험을 제공하려면 Link컴포넌트가 필요합니다.

```
import Link from "next/link";
import { useRouter } from "next/router";

export default function NavBar() {
  const router = useRouter();
  return (
    <nav>
      <Link href="/">
        <a style={{ color: router.pathname === "/" ? "red" : "blue" }}>Home</a>
      </Link>
      <Link href="/about">
        <a style={{ color: router.pathname === "/about" ? "red" : "blue" }}>
          About
        </a>
      </Link>
    </nav>
  );
}
```

# useRouter

- 앱의 함수 컴포넌트에서 router객체 내부에 접근하려면 userRouter()훅을 사용할 수 있습니다.
- useRouter는 React Hook입니다. 즉, 클래스와 함께 사용할 수 없습니다.
- withRouter를 사용하거나 클래스를 함수 컴포넌트로 래핑할 수 있습니다.

```
import Link from "next/link";
import { useRouter } from "next/router";

export default function NavBar() {
  const router = useRouter();
  return (
    <nav>
      <Link href="/">
        <a style={{ color: router.pathname === "/" ? "red" : "blue" }}>Home</a>
      </Link>
      <Link href="/about">
        <a style={{ color: router.pathname === "/about" ? "red" : "blue" }}>
          About
        </a>
      </Link>
    </nav>
  );
}
```

## CSS modules (Styles JSX가 갠적으로 더 편함)

- css modules는 우리가 평범한 css를 사용할 수 있도록 도와준다.
- 클래스를 추가할 때 클래스이름을 텍스트로서 추가하지 않는다.
- 자바스크립트에서의 프로퍼티 형식으로 추가한다.
- 이러한 접근방식의 장점은 어떤 '충돌'도 일어나지 않는다. (다른 컴포넌트에서 같은 이름의 클래스이름을 사용할 수 있다.)

```
// NabBar.module.css 의 코드
.link {
  text-decoration: none;
}
```

```
// 개발자도구에서의 코드
// link 클래스의 이름이 특이하게 되어있다.
<a class="NavBar_link__m_ckV NavBar_active__tG_1I" href="/">Home</a>
```

# Styles JSX

- NextJS의 고유한 style추가방법이다.
- style jsx는 해당 파일에서만 클래스를 불러올 수 있기때문에 다른 파일에 영향을 주지 않는다.

```
import Link from "next/link";
import { useRouter } from "next/router";

export default function NavBar() {
  const router = useRouter();
  return (
    <nav>
      <Link href="/">
        <a className={router.pathname === "/" ? "active" : ""}>Home</a>
      </Link>
      <Link href="/about">
        <a className={router.pathname === "/about" ? "active" : ""}>About</a>
      </Link>
      <style jsx>{`
        nav {
          background-color: tomato;
        }
        a {
          text-decoration: none;
        }
        .active {
          color: yellow;
        }
      `}</style>
    </nav>
  );
}
```

# Global style

- \_app.js라는 파일을 만든다. (반드시 이 이름이어야 한다.! 왜냐면 NextJS는 다른page가 렌더링되기전에 app파일을 먼저 보기 때문이다.)
- next는 app()함수에 2가지 prop을 가져옵니다. (Componets, pageProps => 컴포넌트, 불러올 페이지)
- next는 global.css를 import 할 수 없습니다.
- global style을 여기서 만들 수 있습니다.

```
import NavBar from "../components/NavBar";
import "../styles/globals.css";

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <NavBar />
      <Component {...pageProps} />
    </>
  );
}
```
