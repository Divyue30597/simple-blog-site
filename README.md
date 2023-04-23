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

## Code splitting and prefetching

- Next.js does the code splitting automattically. It is like _what you want is what you get_. Rendering the home page will load the code required for that page, while the code for the other pages is not served ensuring that home page loads quickly. Only loading the code for the page you request also means that pages become isolated and if certain page throws error, the rest of the app will work fine.
- And while using the `Link` tag, the pages are **prefetched** on the background automatically in the production build.

## `next/images`

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

## Title of the page

- We can modify the title of the page, using the `next/head`, which has the next react component, which updates the document title of the page.

```js
import Head from 'next/head';

...
    <Head>
        <title>This is the title of the document</title>
    </Head>
...
```

## Script tag

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

## Styling the next application

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

### Global Styles

Global styles are added to the application, which is applied only once to the whole application once and **should not** be reused else where since it affects all the element on the page. Start by adding `_app.js` file to the `pages` directory. Create a `globals.css` file in the `styles` folder at the root level of the application.

The style written in the `globals.css` file will be applied to the whole application.

```js
import "../styles/globals.css";

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

**PS:** Restart the server after adding the `_app.js` to the application.

## Pre-rendering

- Next.js pre-renders every page. This means that Next.js _generates HTML for each page in advance_, instead of having it all done by client-side JavaScript.
- Each generated HTML is associated with minimal JavaScript code necessary for that page. When a page is loaded by the browser, its JavaScript code runs and makes the page fully interactive. (This process is called hydration.)
- if you use plain react-app, then there is no pre-rendering, it is exclusive to next.js

1. **Initial loads:** pre-rendered HTML is displayed.
2. JS Loads
3. **Hydration:** React components are initalized and app becomes interactive

### Two forms of pre-rendering

The difference is **when** it generated the HTML for the page.

- **Static Generation:** generates the HTML at the **build time**, it is then **reused** on each request.

- **Server-side Generation:** generates the HTML on **each request**.

#### When to use Static generation vs Server-side rendering

- Using Static generation is recommended wherever possible because your page can be built once and served by CDN, which makes it much faster than having a server render the page on every request.
- You should ask yourself: "Can I pre-render this page ahead of a user's request?" If the answer is yes, then you should choose Static Generation.

### Static generation without data

Pages that donot require fetching external data will automatically statically generated when the app is build for production.

### Static generation with data

For some pages, we might need to fetch the data from the external file system, external API or query the database at the build time, thus rendering the HTML without first fetching the data might not be possible.

Now, In a page component, you can export an `async` function called `getStaticProps`.

- `getStaticProps` runs at the build time and
- Inside the function, you can fetch external data and send it as props to the page.

```js
import fs from "fs";
import path from "path";
import matter from "gray-matter";

const postsDirectory = path.join(process.cwd(), "posts");

export function getSortedPostData() {
  // Get filenames under the posts folder
  const fileNames = fs.readdirSync(postsDirectory);

  const allPostsData = fileNames.map((fileName) => {
    // remove .md from the filename
    const id = fileName.replace(/\.md$/, "");

    // Read markdown file as a string
    const fullPath = path.join(postsDirectory, fileName);
    const fileContents = fs.readFileSync(fullPath, "utf8");

    const matterResult = matter(fileContents);

    return {
      id,
      ...matterResult.data,
    };
  });

  return allPostsData.sort((a, b) => {
    if (a.date < b.date) {
      return 1;
    } else {
      return -1;
    }
  });
}
```

### `getStaticProps`

- Can be used to fetch external API or Query database directly.

```js
export async function getSortedPostsData() {
  const res = await fetch("...");
  return res.json();
}
```

- Query from the database directly.

```js
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```

**NOTES:** `getStaticProps` only runs on the **server-side props**. It will **never run on the client-side**. It won’t even be included in the JS bundle for the browser. That means you can write code such as direct database queries without them being sent to browsers.

- In **development** (`npm run dev`), `getStaticProps` runs on _every_ requests.
- In **production**, `getStaticProps` runs at build time.

### `getServerSideProps`

When you need to fetch the data at the **request time**. Context contains request specific parameters.

```js
export async function getServerSideProps(context) {
  return {
    props:{
      ...
    }
  }
}
```

#### Client-side rendering strategy

If you do not need to pre-render the data, you can also use the following strategy (called Client-side Rendering):

- Statically generate (pre-render) parts of the page that do not require external data.
- When the page loads, fetch external data from the client using JavaScript and populate the remaining parts.

## Statically generate pages with dynamic Routes

Page path depends upon the external path in case of data coming from external source. Using dynamic URL concept, next.js allows to statically generate pages with paths that depend on the external data.

- `path/<id>` -> path format.
- export `getStaticPaths` from the Layout page and return the list of id from this page.
- we need to implement `getStaticProps`, to fetch the necessary data with given id.

```js
import Layout from "../../components/layout";

export default function Post() {
  return <Layout>...</Layout>;
}

export async function getStaticPaths() {
  // Return a list of possible value for id
  return {
    Paths,
    fallback: false,
  };
}

export async function getStaticProps({ params }) {
  // Fetch necessary data for the blog post using params.id
}
```

### Fallback in `getStaticPaths`

`fallback: false`:

- returns **404** page, if any path is not returned by `getStaticPath`

`fallback:true`:

- paths returned by `getStaticPath` will be rendered to HTML at the build time.
- Paths not generated at the build time will not throw 404 page. Next.js will serve a “fallback” version of the page on the first request to such a path.
- In the background, Next.js will statically generate the requested path. Subsequent requests to the same path will serve the generated page, just like other pages pre-rendered at build time.

`fallback: blocking`:

- new paths will be server-side rendered with getStaticProps, and cached for future requests so it only happens once per path.

[Read-more about dynamic-routes](https://nextjs.org/learn/basics/dynamic-routes/dynamic-routes-details)
