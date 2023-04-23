# Notes for Next.js, a react framework

## Routing

Next.js has a file-system based router built on the concept of pages. When a file is added to the pages directory, it's automatically available as a route. The files inside the pages directory can be used to define most common patterns.

- `pages/index.js` -> `/`
- `pages/blog/index.js` -> `/blog`

### `Link` tag

- Link is used for client side navigation between two pages on next application.
- Client side navigation means that page transition happens because of the `Javascript`, which is faster than the default navigation behaviour of the browser.
- To test it
  - add `backgrount-color` CSS property to html and click on the `Link` we can see that the color will persist on both the pages and this proves Client Side navigation works fine.
- Using `<a href="">` means that the page will refresh and the backgound-color will be cleared.

### Code splitting and prefetching

- Next.js does the code splitting automattically. It is like _what you want is what you get_. Rendering the home page will load the code required for that page, while the code for the other pages is not served ensuring that home page loads quickly. Only loading the code for the page you request also means that pages become isolated and if certain page throws error, the rest of the app will work fine.
- And while using the `Link` tag, the pages are **prefetched** on the background automatically in the production build.

### `next/images`

- Next.js supports image optimization such as resizing, optimizing, and serving images in modern formats, and `next/images` is an extension to the existing `<img />` element.
- Instead of optimizing images at build time, Next.js optimizes images on-demand, as users request them. Unlike static site generators and static-only solutions, your build times aren't increased, whether shipping 10 images or 10 million images.
- Images are lazy loaded by default. That means your page speed isn't penalized for images outside the viewport. Images load as they are scrolled into viewport.

```js
import Images from "next/image";

<Images
  src="/images/profile_picture.jpg"
  height={144}
  width={144}
  alt="Divyue Sharma"
/>;
```

### Title of the page

- We can modify the title of the page, using the `next/head`, which has the next react component, which updates the document title of the page.

```js
import Head from 'next/head';

...
    <Head>
        <title>This is the title of the document</title>
    </Head>
...
```

### Script tag

Loading `script` in the `Head` component does not give the clear when that script will load with respect to other js code fetched on the same page. This can be solved using the `next/script`, which is an extension to the HTML script component.

This should not be present in the `Head` component.

```js
import Script from "next/script";

...
<Script
  src="https://connect.facebook.net/en_US/sdk.js"
  strategy="lazyOnload"
  onLoad={() =>
    console.log(`script loaded correctly, window.FB has been populated`)
  }
/>
...
```

- `strategy` controls when the third-party script should load. A value of lazyOnload tells Next.js to load this particular script lazily during browser idle time.
- `onLoad` is used to run any JavaScript code immediately after the script has finished loading. In this example, we log a message to the console that mentions that the script has loaded correctly.

### Styling the next application

- This can be done using the `CSS module`, which allows to locally scope CSS at the component level, by automatically creating unique class names.

```js
export default function Layout({ children }) {
  return <div>{children}</div>;
}
```

CSS file `layout.module.css`

```css
.container {
  max-width: 36rem;
  padding: 0 1rem;
  margin: 3rem auto 6rem;
}
```

Updated Layout component

```js
import styles from "./layout.module.css";

export default function Layout({ children }) {
  return <div className={styles.container}>{children}</div>;
}
```

The `className` will be generating a unique className string which is available to this particular component thus applying the style to this component alone. This is done by next.

Code Splitting feature works here. So bundled size is reduced and minimal amount of CSS is loaded for each page.

#### Global Styles

Global styles are added to the application, which is applied only once to the whole application once and **should not** be reused else where since it affects all the element on the page. Start by adding `_app.js` file to the `pages` directory. Create a `globals.css` file in the `styles` folder at the root level of the application.

The style written in the `globals.css` file will be applied to the whole application.

```js
import "../styles/globals.css";

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

**PS:** Restart the server after adding the `_app.js` to the application.
