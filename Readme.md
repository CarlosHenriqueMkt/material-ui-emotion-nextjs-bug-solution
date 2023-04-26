# Incompatibility between Material UI, Emotion, and Next.js SSR

**Issue Description:**
When using Material UI and Emotion together in a Next.js application, you may encounter a compatibility issue that prevents server-side rendering (SSR) of styles. This can result in a flicker of unstyled content when loading the page, which can be especially noticeable on slower connections. The issue arises due to the way Material UI and Emotion generate CSS class names, leading to conflicting class names between the two libraries.

**Technical Details:**
Material UI and Emotion use different strategies for generating CSS class names. Material UI uses a deterministic class name generator that relies on the component's name and a unique key, whereas Emotion uses a non-deterministic class name generator that generates unique class names for each rendered instance of a component. When Material UI and Emotion are used together in a Next.js application, they may generate conflicting class names that can result in missing or incorrect styles during server-side rendering.

**Solution:**
The solution for this problem is to use the "@emotion/cache" plugin of Emotion and pass it to the Material UI ThemeProvider. This will allow Material UI and Emotion to share the same style cache and resolve the generated class conflict.

To implement this solution, you will need to install "@emotion/cache" and "@emotion/react" in your Next.js project. You will also need to modify the "_document.js" file in the "pages" folder of your project to import the necessary packages and create a cache provider. Finally, you will need to wrap your component with the Emotion "CacheProvider" in your "_app.js" file.

Once implemented, this solution should resolve the compatibility issue between Material UI, Emotion, and Next.js SSR, allowing you to render styles correctly on the server and prevent unstyled content flicker on the client.

## Here is an example of how to do it:

**Step 1:**
Install the @emotion/cache and @emotion/react packages in your Next.js project using the following command:

```npm install @emotion/cache @emotion/react```
or
`` yarn add @emotion/cache @emotion/react```

**Step 2:**
Change the _document.js file in the pages folder of your Next.js project and add the following code:

```
import createCache from "@emotion/cache";
import { CacheProvider } from "@emotion/react";
import Document, { Head, Html, Main, NextScript } from "next/document";
import { ServerStyleSheets } from "@material-ui/core/styles";

export default function MyDocument() {
  return (
    <Html>
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}

MyDocument.getServerSideProps = async (ctx) => {
  //Creates Emotion new cache
  const cache = createCache({ key: "css" });
  cache.compat = true;

  // Creates a new instance of ServerStyleSheets for the Material-UI
  const sheets = new ServerStyleSheets();

  const originalRenderPage = ctx.renderPage;

  // Renders the page with Emotion and Material-UI
  ctx.renderPage = () =>
    originalRenderPage({
      enhanceApp: (App) => (props) =>
        (
          <CacheProvider value={cache}>
            {sheets.collect(<App {...props} />)}
          </CacheProvider>
        ),
    });

  // Fetches the initial props of the page using the Document's default getInitialProps
  const initialProps = await Document.getInitialProps(ctx);

  // Adds the Material-UI style collected by ServerStyleSheets to the styles property of the return object
  return {
    ...initialProps,
    styles: (
      <>
        {initialProps.styles}
        {sheets.getStyleElement()}
      </>
    ),
  };
};
```

**Step 3:**
Wrap the Material UI ThemeProvider component with the Emotion CacheProvider in your _app.js file using the following code:

```
import { CacheProvider } from "@emotion/react";
import { ThemeProvider } from "@material-ui/core/styles";
import CssBaseline from "@material-ui/core/CssBaseline";
import createCache from "@emotion/cache";
import { useEffect } from "react";
import theme from "@/theme";

// Create a new emotion cache
const cache = createCache({ key: "css" });
cache.compat = true;

function MyApp({ Component, pageProps }) {
    // Remove the MUI style added by the server
  useEffect(() => {
    const jssStyles = document.querySelector("#jss-server-side");
    if (jssStyles) {
      jssStyles.parentElement.removeChild(jssStyles);
    }
  }, []);

  return (
    // Provides the Emotion cache for the application
    <CacheProvider value={cache}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        <Component {...pageProps} />
      </ThemeProvider>
    </CacheProvider>
  );
}

export default MyApp;
```

Note that you need to import correctly your Material UI theme.

**Conclusion:**
These steps should help resolve the issue of incompatibility between Material UI, Emotion, and Next.js SSR.
