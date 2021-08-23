

## 資源及連結

基本上官網就寫得蠻清楚的

https://nextjs.org/docs/getting-started

PJCHENder大的筆記還不錯

https://pjchender.dev/react/nextjs-getting-started



## 安裝

環境

`npm install next react react-dom`

`yarn add next react react-dom`

建立專案

`npx create-next-app`



## SSR & SSG

Next.js 可以 `SSR`及 `SSG` 混合使用，Nuxt.js 好像只能則一的樣子（不是很確定）

SSR -> Server Side Rendering

SSG -> Static Site Generation

CSR -> Client Side Rendering

關於（v9.3 版後） SSR 和 SSG 混合式開發的介紹

https://www.ithome.com.tw/news/136271

官網對這兩種 pre-rendering型式的解釋

https://nextjs.org/docs/basic-features/pages#two-forms-of-pre-rendering

Next.js的 `SSG`有額外處理的方法，之前學 Nuxt.js的時候似乎沒有這方面的概念（或者是其實也有只是我們之前看的那個教學影片沒教？），但這個東西在 Next.js裡也是9.3版之後才有似乎是比較新的東西



## Data Fetching

這篇文章說明的算是蠻清楚的

https://pjchender.dev/react/nextjs-getting-started/#pre-rendering-and-data-fetching

### getInitialProps

我的理解這個就類似 Nuxt.js的 `asyncData`，也就是 SSR的的開發策略：同一份code同時在 server-side和 client-side執行（也就是所謂同構(**Universal Rendering** / **Isomorphic render**)）

**初次載入的時候是在 server-side執行，透過前端router進入時是在 client-side執行。**

在載入頁面之前非同步取得資料，並將資料變為該 Page Component的 `props`（用 return的方式）

**BUT** 官方建議使用 `getStaticProps`和 `getServerSideProps`取代 `getInitialProps` (https://nextjs.org/docs/api-reference/data-fetching/getInitialProps)

官網上這樣寫讓人有些困惑，在 Nuxt.js中大量使用的方法， Next.js竟然建議不要使用！？

* **我的理解**

官方的意思可能是說應該「優先」考慮採用 SSG 的方式建立靜態頁面（效能較好？https://nextjs.org/docs/basic-features/pages#when-should-i-use-static-generation），如果有明確的需求（比如動態取得渲染所需的資料、又有前後端同構的需求），才使用 SSR（而不是說不要採用 SSR）。也就是說這兩個是可以混用的，只是以 SSG為優先而已。

但我覺得這樣子的說法對初學者來說很可能被誤導，就我自己的實務經驗不同的系統有不同的需求，以我工作的需求來說 data fetching幾乎都是必須 run-time取得的，`getInitialProps`對我來說還是有使用的必要

（內心疑惑：或者Next.js根本就認為 run-time取得的資料沒有 SEO的必要、而有 SEO必要的資料不應該 run-time取得？？？）。

* **要注意的許多坑！！**

1. `getInitialProps`如果使用在最外層(`app.js`)則只會在 server-side調用（喔…）

2. 如果使用在最外層(`app.js`)，則注入的 `context` 參數會放在 `{ ctx }`  中，和使用在 page時不同（！！）

   ```js
   // PAGE
   function getInitialProps (context) {
     const { req, res }  = context
     return {}
   }
   // app.js
   function getInitialProps ({ Component, ctx }) {
     const { req, res }  = ctx
     return {}
   }
   ```



3. 如果在最外層(`app.js`)使用`getInitialProps`，則 Page也同時使用`getInitialProps`的話，最外層(`app.js`)的優先度是最高的，只有最外層會被調用，Page的 `getInitialProps`是不會被調用的。如果要調用 Page的 `getInitialProps`要這樣寫：

   ```js
   // app.js
   function getInitialProps ({ Component, ctx }) {
     const { req, res }  = ctx
     Component.getInitialProps({ req, res }) // 可呼叫 Page的 getInitialProps
     return {}
   }
   ```

   

4. 如果 Page是巢狀結構（nested routes）的話，內外層Page都同時有 `getInitialProps`，則只有最內層的會被調用，如果希望同時都能被調用的話，要這樣寫：

   ```js
   // child page
   import ParentPage from '@/pages/parentpage/index'
   
   function getInitialProps (context) {
     const { data } = ParentPage.getInitialProps(context)
     return { data }
   }
   
   export default function ChildPage ({ data }) {
     return (
       <!-- 這裡要注意… 將父層的 getInitialProps的回傳值傳入父層的 Component -->
     	<ParentPage data={data}>
       	<div>
         </div>
       </ParentPage>
     )
   }
   ```

   但如同前面所述，`app.js`的`getInitialProps`的優先度仍然是最高的



### getServerSideProps

跟 `getInitialProps`差異是他只能在 server side執行，並且在 page 中使用`getServerSideProps` 就能使用 `getInitialProps` （只能二選一）

如果非必要在 server執行就盡量不要使用，官方這樣說明：

> You should use getServerSideProps only if you need to pre-render a page whose data must be fetched at request time. Time to first byte (TTFB) will be slower than getStaticProps because the server must compute the result on every request, and the result cannot be cached by a CDN without extra configuration.
> If you don’t need to pre-render the data, then you should consider fetching data on the client side.

```js
import fetch from "isomorphic-unfetch";
function Page({ data }) {
  // Render data...
}
export async function getServerSideProps() {
  const res = await fetch(`https://.../data`);
  const data = await res.json();
  return { props: { data } };
}
export default Page;
```





### getStaticProps, getStaticPaths

用來處理 `SSG` (Static Site Generation) 的需求，用來建立靜態頁面，在 **build-time**的時候執行**（但是在開發模式(development mode)下都會採用SSR的方式，也就是run-time的時候執行**





## Context

`getInitialProps` 的 Context

https://nextjs.org/docs/api-reference/data-fetching/getInitialProps#context-object

`getServerSideProps` 的 Context

https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering



## Routing

### next/link

等同於 nuxt的 `<nuxt-link>`

```js
import Link from "next/link";
function Home() {
  return (
    <Link href="/">
      <a>Home</a>
    </Link>
  );
}
```

不過經過我的測試 `</Link>`只有 dynamic router的功能，並不會像 nuxt的 `<nuxt-link>`會建立超連結（產生網址在Browser URL bar、並且可按右鍵開新視窗）。如果希望能同時具備 dynamic router以及超連結，可以這樣寫：

```js
<Link href="/">
  <a href="/">Home</a>
</Link>
```



### next/router

等同於 vue/nuxt 的 `$router`

詳細的 api參考官方文件 https://nextjs.org/docs/api-reference/next/router#router-object

```js
import { useRouter } from 'next/router'

function ReadMore() {
  const router = useRouter()

  return (
    <span onClick={() => router.push('/about')}>Click here to read more</span>
  )
}

export default ReadMore
```

參數傳遞和 vue-router有一些不一樣，vue-router的 `query`和 `params`在 react/next中都是 `query`沒有區分開來

比如說 page的路徑及檔名設定為 `pages/post/[pid].js`，路由為 `/post/abc?foo=bar`，則：

```js
import { useRouter } from 'next/router'
const router = useRouter()
console.log(router.query) // { "foo": "bar", "pid": "abc" }
```



### Shallow Routing（淺路由）

官網上對 `Shallow Routing`的解釋為

> Shallow routing allows you to change the URL without running data fetching methods again, that includes [`getServerSideProps`](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering)tps://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation), and [`getInitialProps`](https://nextjs.org/docs/api-reference/data-fetching/getInitialProps)

路由變化的時候不會調用到 data fetching的方法包含 `getServerSideProps`和 `getInitialProps`

**使用方式是要加上 `{ shallow: true }`**

```js
import { useEffect } from 'react'
import { useRouter } from 'next/router'

// Current URL is '/'
function Page() {
  const router = useRouter()

  useEffect(() => {
    // Always do navigations after the first render
    router.push('/?counter=10', undefined, { shallow: true })
  }, [])

  useEffect(() => {
    // The counter changed!
  }, [router.query.counter])
}

export default Page
```



### router.events

當有類似製作middleware程式的需求，也就是在路由改變的事件中所調用的程式，在 vue中稱為 `Navigation Guards`，提供 `beforeRouteEnter` 等監聽事件。而 react也是有提供類似的監聽事件，詳細API參考官方文件 https://nextjs.org/docs/api-reference/next/router#routerevents

```js
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function MyApp({ Component, pageProps }) {
  const router = useRouter()

  useEffect(() => {
    const handleRouteChange = (url, { shallow }) => {
      console.log(
        `App is changing to ${url} ${
          shallow ? 'with' : 'without'
        } shallow routing`
      )
    }

    router.events.on('routeChangeStart', handleRouteChange)

    // If the component is unmounted, unsubscribe
    // from the event with the `off` method:
    return () => {
      router.events.off('routeChangeStart', handleRouteChange)
    }
  }, [])

  return <Component {...pageProps} />
}
```



### redirect

`redirect`指的是網頁在初始化之前就導向到其他的路由，有分為後端及前端兩種

相關教學 https://maxschmitt.me/posts/next-js-redirects/

```js
import Router from 'next/router'
async function getInitialProps({ res, user }) {
    if (!user) {
        if (res) {
            // On the server, we'll use an HTTP response to
            // redirect with the status code of our choice.
            // 307 is for temporary redirects.
            res.writeHead(307, { Location: '/' })
            res.end()
        } else {
            // On the client, we'll use the Router-object
            // from the 'next/router' module.
            Router.replace('/')
        }
        // Return an empty object,
        // otherwise Next.js will throw an error
        return {}
    }
    return { secretData: '...' }
}

```





## Dynamic Import

https://nextjs.org/docs/advanced-features/dynamic-import

```js
import dynamic from 'next/dynamic'
const DynamicComponent = dynamic(() => import('../components/hello'))
function Home() {
  return (
    <div>
      <Header />
      <DynamicComponent />
      <p>HOME PAGE is here!</p>
    </div>
  )
}
export default Home
```



## Layout/嵌套路由 (nest routes)

依我目前的研究，Next.js 預設似乎是沒有 Layout和嵌套路由的概念（不像 Nuxt.js原生就提供製作Layout和嵌套路由的API），必須自己實作出來（竟然！）

### 嵌套路由 (nest routes)

以下網址有教學，雖然裡面使用的名稱是 layout，但是概念上我覺得更像是嵌套路由（至少是接近 Nuxt.js的），然後也有提到 HOC（是比較舊的 design pattern了，畢竟是較早的文章但可以參考看看）

https://ithelp.ithome.com.tw/articles/10190620

方法意外的巧妙，簡單來說，就是使用 Component的 `children`就可以實作出來：

```js
// layout
export default (props) => (
  <div id="main">
    <li><Link href="/about/company">About company</Link></li>
    <li><Link href="/about/me">About me</Link></li>
    {props.children}
  </div>
)
export default About

// child page
import Layout from '../components/layout'
export default () => (
  <Layout>
    <div> 製作一個具有Layout的網站吧 </div>
  </Layout>
)
```



### Layout

參考以下網址的教學，另外也有看到網路上的範例使用同樣的做法。這種寫法和前述嵌套路由的做法很像，但是這個做法的 layout會被固定在最外層，並且內層並不需要將 layout component寫進 function當中

https://dev.to/ozanbolel/layout-persistence-in-next-js-107g

```js
// app.js
import React from "react";
export default function MyApp({ Component, pageProps }) {
  const Layout = Component.Layout ? Component.Layout : React.Fragment;
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  )
}

// layout
import React, { useState } from "react";
export default function MyLayout({ children }) {
  const [counter, setCounter] = useState(0);
  return (
    <>
      <p>      
        <button onClick={() => setCounter(counter + 1)}>
          Clicked {counter} Times
        </button>
      </p>
      {children}
    </>
  )
}

// child page
import Link from "next/link";
import MyLayout from "../layouts/MyLayout";
export default function Profile() {
  return (
    <div>
      <p>This is the Profile page.</p>
      <p>
        <Link href="/account">
          <a>Go: Account</a>
        </Link>
      </p>
    </div>
  );
}
Profile.Layout = MyLayout; // <-- 只要加上這行
```



## CSS樣式

載入CSS檔的規則：

- 全域 css檔必須在 pages/_app.js 檔案中引入（其它地方引入會報錯）
- Component level檔案要以 .module.(css|scss|sass) 命名，可以在任意模組中引入
- Component level檔案已經內建了 scope 功能，每個class將分配一個唯一id，不用擔心命名衝突

參考資料：https://nextjs.org/docs/basic-features/built-in-css-support

因為使用上還是會有些限制，尤其是在使用 SCSS及 Mixins會有些困難，網路上介紹的套件（再找時間研究）：

[@zeit/next-sass](https://www.mdeditor.tw/jump/aHR0cHM6Ly93d3cubnBtanMuY29tL3BhY2thZ2UvQHplaXQvbmV4dC1zYXNz)：解決全域性和模組化問題

[sass-resources-loader](https://www.mdeditor.tw/jump/aHR0cHM6Ly93d3cubnBtanMuY29tL3BhY2thZ2Uvc2Fzcy1yZXNvdXJjZXMtbG9hZGVy)：解決全域性變數和 mixin 問題

介紹網頁（再找時間研究）

https://www.mdeditor.tw/pl/p1n1/zh-tw



### Component-level scss

```js
import styles from './<MyComponent>.module.scss>';
```

使用方式

```jsx
<TheComponent className={styles.someclassselector} />
```

經過我的測試這樣是吃不到樣式的…

```jsx
import './<MyComponent>.module.scss>';

<TheComponent className="someclassselector" />
```

巢狀(nested) scss的使用方式

```js
// scss
.classname{
  .childname{
  }
}

// jsx
<Grid className={css.classname}>
  <Typography className={css.childname}>
  </Typography>
</Grid>
```





## next.config.js

### 根目錄路徑alias

Nuxt預設好像沒有根目錄的路徑寫法（？），所以要自己在 nuxt.config.js加入以下設定，設定好之後就可以用`@`符號來代表根目錄：

`import Button from '@/components/Button.js'`

```js
const path = require("path");
module.exports = {
  webpack: (config) => {
    config.resolve.alias["@"] = path.resolve(__dirname);
    return config;
  },
}
```

https://blog.csdn.net/weixin_40532650/article/details/113119266



### 環境變數

* **基本設定方式：**

第一種做法是建立 `.env`檔，第二種是在 `next.config.js`裡加入

```js
module.exports = {
  env: {
    customKey: 'my-value',
  },
}
```

* **依環境設定方式：**

做法和 nuxt.js差不多，node.js 預設三種開發環境 `development`, `test`, `production`，分別對應以下三種檔名，建在根目錄下：

`.env.production` 

`.env.development`

`.env.test`

取用環境變數寫法 `process.env.[變數名稱]`

環境變數預設只能在 node.js環境中執行，如果要在瀏覽器端上能夠取得的話，變數名稱必須要有前綴 `NEXT_PUBLIC_`

* 自訂環境名稱

next.js 似乎是無法自訂 node.js預設的開發環境以外的名稱，否則會出現以下錯誤：

https://nextjs.org/docs/messages/non-standard-node-env

目前看起來比較好的解法是在 `package.json`的 `script`指令中代入變數，然後在 `next.config.js`裡依變數的名稱切換所需要的變數

比如 script指令如下：

`APP_ENV=staging npm run build`

然後在 config就可以使用 `process.env.APP_ENV`取到 `staging`，再依此取得 .env檔中的變數



## 其他相關文章

Next.js和Nuxt.js的語法比較，Vue和React的兩大SSR解決方案

https://www.mdeditor.tw/pl/pdbQ/zh-tw





