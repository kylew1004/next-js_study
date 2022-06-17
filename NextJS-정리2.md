# Head (next/head)

- 페이지 head에 엘리먼트를 추가하기 위한 내장 컴포넌트를 노출합니다.
  head에 태그가 중복되지 않도록 하려면 다음 예제와 같이 태그가 한 번만 렌더링되도록 하는 key 속성을 사용할 수 있습니다.
  이 경우 두 번째 meta property="og:title"만 렌더링됩니다. 중복 키 속성이 있는 메타 태그는 자동으로 처리됩니다.

# next.config.js

// https://nextjs.org/docs/api-reference/next.config.js/introduction

- Next.js에서 커스텀 설정을 하기 위해서는 프로젝트 디렉터리의 루트(package.json 옆)에 next.config.js 또는 next.config.mjs 파일을 만들 수 있습니다.
- next.config.js는 JSON 파일이 아닌 일반 Node.js 모듈입니다.
- Next.js 서버 및 빌드 단계에서 사용되며 브라우저 빌드에는 포함되지 않습니다.

# redirect 와 rewrite

- redirect (URL이 바뀜)

  - request 경로를 다른 destination 경로로 Redirect할 수 있습니다. Redirect을 사용하려면 next.config.js에서 redirects 키를 사용할 수 있습니다.
  - source, destination 및 permanent 속성이 있는 객체를 포함하는 배열을 반환하는 비동기 함수입니다.
    - source: 들어오는 request 경로 패턴 (request 경로)
    - destination: 라우팅하려는 경로 (redirect할 경로)
    - permanent: true인 경우 클라이언트와 search 엔진에 redirect를 영구적으로 cache하도록 지시하는 308 status code를 사용하고, false인 경우 일시적이고 cache되지 않은 307 status code를 사용합니다.

- rewrites (URL이 안바뀜)

  - Rewrites를 사용하면 들어오는 request 경로를 다른 destination 경로에 매핑할 수 있습니다.
  - Rewrites은 URL 프록시 역할을 하고 destination 경로를 mask하여 사용자가 사이트에서 위치를 변경하지 않은 것처럼 보이게 합니다. 반대로 redirects은 새 페이지로 reroute되고 URL 변경 사항을 표시합니다.

```
//next.config.js 파일
const API_KEY = process.env.API_KEY;

module.exports = {
  reactStrictMode: true,
  async redirects() {
    return [
      {
        source: "/old-blog/:path*",
        destination: "/new-sexy-blog/:path*",
        permanent: false,
      },
    ];
  },
  async rewrites() {
    return [
      {
        source: "/api/movies",
        destination: `https://api.themoviedb.org/3/movie/popular?api_key=${API_KEY}`,
      },
    ];
  },
};

```

# getServerSideProps

- page에서 서버 측 랜더링 함수인 getServerSideProps함수를 export하는 경우 Next.js는 getServerSideProps에서 반환된 데이터를 사용하여 각 request에서 이 페이지를 pre-render합니다.
- getServerSideProps는 서버 측에서만 실행되며 브라우저에서는 실행되지 않습니다.

- 아래 코드는 getServerSideProps를 사용하여 request시 데이터 fetch하기 예시입니다.

```
import { useEffect, useState } from "react";
import Seo from "../components/Seo";

export default function Home({ results }) {
  return (
    <div className="container">
      <Seo title="Home" />
      {results?.map((movie) => (
        <div className="movie" key={movie.id}>
          <img src={`https://image.tmdb.org/t/p/w500/${movie.poster_path}`} />
          <h4>{movie.original_title}</h4>
        </div>
      ))}
      <style jsx>{`
        .container {
          display: grid;
          grid-template-columns: 1fr 1fr;
          padding: 20px;
          gap: 20px;
        }
        .movie img {
          max-width: 100%;
          border-radius: 12px;
          transition: transform 0.2s ease-in-out;
          box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 12px;
        }
        .movie:hover img {
          transform: scale(1.05) translateY(-10px);
        }
        .movie h4 {
          font-size: 18px;
          text-align: center;
        }
      `}</style>
    </div>
  );
}

export async function getServerSideProps() {
  const { results } = await (
    await fetch(`http://localhost:3000/api/movies`)
  ).json();
  return {
    props: {
      results,
    },
  };
}
```

## getServerSideProps를 언제 사용하는게 좋을까?

- request time에 반드시 데이터를 fetch해와야 하는 페이지를 pre-render해야 하는 경우에만 getServerSideProps를 사용해야 합니다.
- 데이터를 pre-render할 필요가 없다면 client side에서 데이터를 가져오는 것을 고려해야 합니다.

# Dynamic Routes

- Next.js에서는 page에 대괄호([param])를 추가하여 Dynamic Route를 생성할 수 있습니다.
  - params: 이 페이지에서 dynamic route(동적 경로)를 사용하는 경우 params에 route parameter가 포함됩니다. 페이지 이름이 `[id].js`이면 params는 { id: ... }처럼 보일 것입니다.
- /movies/1, /movies/abc 등과 같은 모든 경로는 pages/movies/[id].js와 일치합니다.

# Catch all routes

- 대괄호 안에 세 개의 점(...)을 추가하여 모든 경로를 포착하도록 Dynamic Routes를 확장할 수 있습니다. pages/movies/[...id].js는 /movies/1와 일치하지만 /movies/1/2, /movies/1/ab/cd 등과도 일치합니다.
- 일치하는 매개변수는 페이지에 쿼리 매개변수로 전송되며 항상 배열이므로 /movies/a 경로에는 다음 쿼리 개체가 있습니다.

```
import Seo from "../../components/Seo";
import { useRouter } from "next/router";

export default function Detail({ params }) {
  const router = useRouter();
  const [title, id] = params || [];
  return (
    <div>
      <Seo title={title} />
      <h4>{title}</h4>
    </div>
  );
}

export function getServerSideProps({ params: { params } }) {
  return {
    props: {
      params,
    },
  };
}
```

# router.push(url, as, options)

- 클라이언트 측 전환을 처리합니다. 이 방법은 next/link가 충분하지 않은 경우에 유용합니다.
  - url: UrlObject | String: 탐색할 URL
  - as: UrlObject | String: 브라우저 URL 표시줄에 표시될 경로에 대한 선택적 데코레이터입니다.

```
router.push({
pathname: '/post/[pid]',
query: { pid: post.id },
})
```
