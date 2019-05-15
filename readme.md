Next.js
NPM version Build Status Build Status Coverage Status Join the community on Spectrum

Visit nextjs.org/learn to get started with Next.js.

The below readme is the documentation for the canary (prerelease) branch. To view the documentation for the latest stable Next.js version visit nextjs.org/docs

How to use
Setup
Automatic code splitting
CSS
Built-in CSS support
CSS-in-JS
Importing CSS / Sass / Less / Stylus files
Static file serving (e.g.: images)
Populating <head>
Fetching data and component lifecycle
Routing
With <Link>
With URL object
Replace instead of push url
Using a component that supports onClick
Forcing the Link to expose href to its child
Disabling the scroll changes to top on page
Imperatively
Intercepting popstate
With URL object
Router Events
Shallow Routing
Using a Higher Order Component
Prefetching Pages
With <Link>
Imperatively
Custom server and routing
Disabling file-system routing
Dynamic assetPrefix
Dynamic Import
1. Basic Usage (Also does SSR)
2. With Custom Loading Component
3. With No SSR
4. With Multiple Modules At Once
Custom <App>
Custom <Document>
Customizing renderPage
Custom error handling
Reusing the built-in error page
Custom configuration
Setting a custom build directory
Disabling etag generation
Configuring the onDemandEntries
Configuring extensions looked for when resolving pages in pages
Configuring the build ID
Configuring Next process script
Customizing webpack config
Customizing babel config
Exposing configuration to the server / client side
Starting the server on alternative hostname
CDN support with Asset Prefix
Production deployment
Serverless deployment
One Level Lower
Summary
Browser support
Static HTML export
Usage
Copying custom files
Limitation
Multi Zones
How to define a zone
How to merge them
Recipes
FAQ
Contributing
Authors
How to use
Setup
Install it:

npm install --save next react react-dom
and add a script to your package.json like this:

{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
After that, the file-system is the main API. Every .js file becomes a route that gets automatically processed and rendered.

Populate ./pages/index.js inside your project:

function Home() {
  return <div>Welcome to next.js!</div>
}

export default Home
and then just run npm run dev and go to http://localhost:3000. To use another port, you can run npm run dev -- -p <your port here>.

So far, we get:

Automatic transpilation and bundling (with webpack and babel)
Hot code reloading
Server rendering and indexing of ./pages
Static file serving. ./static/ is mapped to /static/ (given you create a ./static/ directory inside your project)
To see how simple this is, check out the sample app - nextgram

Automatic code splitting
Every import you declare gets bundled and served with each page. That means pages never load unnecessary code!

import cowsay from 'cowsay-browser'

function CowsayHi() {
  return (
    <pre>
      {cowsay.say({ text: 'hi there!' })}
    </pre>
  )
}

export default CowsayHi
CSS
Built-in CSS support
Examples
We bundle styled-jsx to provide support for isolated scoped CSS. The aim is to support "shadow CSS" similar to Web Components, which unfortunately do not support server-rendering and are JS-only.

function HelloWorld() {
  return (
    <div>
      Hello world
      <p>scoped!</p>
      <style jsx>{`
        p {
          color: blue;
        }
        div {
          background: red;
        }
        @media (max-width: 600px) {
          div {
            background: blue;
          }
        }
      `}</style>
      <style global jsx>{`
        body {
          background: black;
        }
      `}</style>
    </div>
  )
}

export default HelloWorld
Please see the styled-jsx documentation for more examples.

CSS-in-JS
 Examples
It's possible to use any existing CSS-in-JS solution. The simplest one is inline styles:

function HiThere() {
  return <p style={{ color: 'red' }}>hi there</p>
}

export default HiThere
To use more sophisticated CSS-in-JS solutions, you typically have to implement style flushing for server-side rendering. We enable this by allowing you to define your own custom <Document> component that wraps each page.

Importing CSS / Sass / Less / Stylus files
To support importing .css, .scss, .less or .styl files you can use these modules, which configure sensible defaults for server rendered applications.

@zeit/next-css
@zeit/next-sass
@zeit/next-less
@zeit/next-stylus
Static file serving (e.g.: images)
Create a folder called static in your project root directory. From your code you can then reference those files with /static/ URLs:

function MyImage() {
  return <img src="/static/my-image.png" alt="my image" />
}

export default MyImage
Note: Don't name the static directory anything else. The name is required and is the only directory that Next.js uses for serving static assets.

Populating <head>
Examples
We expose a built-in component for appending elements to the <head> of the page.

import Head from 'next/head'

function IndexPage() {
  return (
    <div>
      <Head>
        <title>My page title</title>
        <meta name="viewport" content="initial-scale=1.0, width=device-width" />
      </Head>
      <p>Hello world!</p>
    </div>
  )
}

export default IndexPage
To avoid duplicate tags in your <head> you can use the key property, which will make sure the tag is only rendered once:

import Head from 'next/head'

function IndexPage() {
  return (
    <div>
      <Head>
        <title>My page title</title>
        <meta name="viewport" content="initial-scale=1.0, width=device-width" key="viewport" />
      </Head>
      <Head>
        <meta name="viewport" content="initial-scale=1.2, width=device-width" key="viewport" />
      </Head>
      <p>Hello world!</p>
    </div>
  )
}

export default IndexPage
In this case only the second <meta name="viewport" /> is rendered.

Note: The contents of <head> get cleared upon unmounting the component, so make sure each page completely defines what it needs in <head>, without making assumptions about what other pages added

Note: <title> and <meta> elements need to be contained as direct children of the <Head> element, or wrapped into maximum one level of <React.Fragment>, otherwise the metatags won't be correctly picked up on clientside navigation.

Fetching data and component lifecycle
Examples
When you need state, lifecycle hooks or initial data population you can export a React.Component (instead of a stateless function, like shown above):

import React from 'react'

class HelloUA extends React.Component {
  static async getInitialProps({ req }) {
    const userAgent = req ? req.headers['user-agent'] : navigator.userAgent
    return { userAgent }
  }

  render() {
    return (
      <div>
        Hello World {this.props.userAgent}
      </div>
    )
  }
}

export default HelloUA
Notice that to load data when the page loads, we use getInitialProps which is an async static method. It can asynchronously fetch anything that resolves to a JavaScript plain Object, which populates props.

Data returned from getInitialProps is serialized when server rendering, similar to a JSON.stringify. Make sure the returned object from getInitialProps is a plain Object and not using Date, Map or Set.

For the initial page load, getInitialProps will execute on the server only. getInitialProps will only be executed on the client when navigating to a different route via the Link component or using the routing APIs.

Note: getInitialProps can not be used in children components. Only in pages.


If you are using some server only modules inside getInitialProps, make sure to import them properly. Otherwise, it'll slow down your app.


You can also define the getInitialProps lifecycle method for stateless components:

function Page({ stars }) {
  return <div>Next stars: {stars}</div>
}

Page.getInitialProps = async ({ req }) => {
  const res = await fetch('https://api.github.com/repos/zeit/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default Page
getInitialProps receives a context object with the following properties:

pathname - path section of URL
query - query string section of URL parsed as an object
asPath - String of the actual path (including the query) shows in the browser
req - HTTP request object (server only)
res - HTTP response object (server only)
jsonPageRes - Fetch Response object (client only)
err - Error object if any error is encountered during the rendering
Routing
Next.js does not ship a routes manifest with every possible route in the application, so the current page is not aware of any other pages on the client side. All subsequent routes get lazy-loaded, for scalability sake.

With <Link>
Examples
Client-side transitions between routes can be enabled via a <Link> component.

Basic Example

Consider these two pages:

// pages/index.js
import Link from 'next/link'

function Home() {
  return (
    <div>
      Click{' '}
      <Link href="/about">
        <a>here</a>
      </Link>{' '}
      to read more
    </div>
  )
}

export default Home
// pages/about.js
function About() {
  return <p>Welcome to About!</p>
}

export default About
Custom routes (using props from URL)

<Link> component has two main props:

href: the path inside pages directory + query string.
as: the path that will be rendered in the browser URL bar.
Example:

Consider you have the URL /post/:slug.

You created the pages/post.js

class Post extends React.Component {
  static async getInitialProps({query}) {
    console.log('SLUG', query.slug)
    return {}
  }
  render() {
    return <h1>My blog post</h1>
  }
}

export default Post
You add the route to express (or any other server) on server.js file (this is only for SSR). This will route the url /post/:slug to pages/post.js and provide slug as part of query in getInitialProps.

server.get("/post/:slug", (req, res) => {
  return app.render(req, res, "/post", { slug: req.params.slug })
})
For client side routing, use next/link:

<Link href="/post?slug=something" as="/post/something">
Note: use <Link prefetch> for maximum performance, to link and prefetch in the background at the same time

Client-side routing behaves exactly like the browser:

The component is fetched
If it defines getInitialProps, data is fetched. If an error occurs, _error.js is rendered
After 1 and 2 complete, pushState is performed and the new component is rendered
To inject the pathname, query or asPath in your component, you can use withRouter.

With URL object
Examples
The component <Link> can also receive an URL object and it will automatically format it to create the URL string.

// pages/index.js
import Link from 'next/link'

function Home() {
  return (
    <div>
      Click{' '}
      <Link href={{ pathname: '/about', query: { name: 'Zeit' } }}>
        <a>here</a>
      </Link>{' '}
      to read more
    </div>
  )
}

export default Home
That will generate the URL string /about?name=Zeit, you can use every property as defined in the Node.js URL module documentation.

Replace instead of push url
The default behaviour for the <Link> component is to push a new url into the stack. You can use the replace prop to prevent adding a new entry.

// pages/index.js
import Link from 'next/link'

function Home() {
  return (
    <div>
      Click{' '}
      <Link href="/about" replace>
        <a>here</a>
      </Link>{' '}
      to read more
    </div>
  )
}

export default Home
Using a component that supports onClick
<Link> supports any component that supports the onClick event. In case you don't provide an <a> tag, it will only add the onClick event handler and won't pass the href property.

// pages/index.js
import Link from 'next/link'

function Home() {
  return (
    <div>
      Click{' '}
      <Link href="/about">
        <img src="/static/image.png" alt="image" />
      </Link>
    </div>
  )
}

export default Home
Forcing the Link to expose href to its child
If child is an <a> tag and doesn't have a href attribute we specify it so that the repetition is not needed by the user. However, sometimes, you�ll want to pass an <a> tag inside of a wrapper and the Link won�t recognize it as a hyperlink, and, consequently, won�t transfer its href to the child. In cases like that, you should define a boolean passHref property to the Link, forcing it to expose its href property to the child.

Please note: using a tag other than a and failing to pass passHref may result in links that appear to navigate correctly, but, when being crawled by search engines, will not be recognized as links (owing to the lack of href attribute). This may result in negative effects on your sites SEO.

import Link from 'next/link'
import Unexpected_A from 'third-library'

function NavLink({ href, name }) {
  return (
    <Link href={href} passHref>
      <Unexpected_A>
        {name}
      </Unexpected_A>
    </Link>
  )
}

export default NavLink
Disabling the scroll changes to top on page
The default behaviour of <Link> is to scroll to the top of the page. When there is a hash defined it will scroll to the specific id, just like a normal <a> tag. To prevent scrolling to the top / hash scroll={false} can be added to <Link>:

<Link scroll={false} href="/?counter=10"><a>Disables scrolling</a></Link>
<Link href="/?counter=10"><a>Changes with scrolling to top</a></Link>
Imperatively
Examples
You can also do client-side page transitions using the next/router

import Router from 'next/router'

function ReadMore() {
  return (
    <div>
      Click <span onClick={() => Router.push('/about')}>here</span> to read more
    </div>
  )
}

export default ReadMore
Intercepting popstate
In some cases (for example, if using a custom router), you may wish to listen to popstate and react before the router acts on it. For example, you could use this to manipulate the request, or force an SSR refresh.

import Router from 'next/router'

Router.beforePopState(({ url, as, options }) => {
  // I only want to allow these two routes!
  if (as !== "/" || as !== "/other") {
    // Have SSR render bad routes as a 404.
    window.location.href = as
    return false
  }

  return true
});
If the function you pass into beforePopState returns false, Router will not handle popstate; you'll be responsible for handling it, in that case. See Disabling File-System Routing.

Above Router object comes with the following API:

route - String of the current route
pathname - String of the current path excluding the query string
query - Object with the parsed query string. Defaults to {}
asPath - String of the actual path (including the query) shows in the browser
push(url, as=url) - performs a pushState call with the given url
replace(url, as=url) - performs a replaceState call with the given url
beforePopState(cb=function) - intercept popstate before router processes the event.
The second as parameter for push and replace is an optional decoration of the URL. Useful if you configured custom routes on the server.

With URL object
You can use an URL object the same way you use it in a <Link> component to push and replace an URL.

import Router from 'next/router'

const handler = () => {
  Router.push({
    pathname: '/about',
    query: { name: 'Zeit' }
  })
}

function ReadMore() {
    return (
    <div>
      Click <span onClick={handler}>here</span> to read more
    </div>
  )
}

export default ReadMore
This uses the same exact parameters as in the <Link> component.

Router Events
You can also listen to different events happening inside the Router. Here's a list of supported events:

routeChangeStart(url) - Fires when a route starts to change
routeChangeComplete(url) - Fires when a route changed completely
routeChangeError(err, url) - Fires when there's an error when changing routes
beforeHistoryChange(url) - Fires just before changing the browser's history
hashChangeStart(url) - Fires when the hash will change but not the page
hashChangeComplete(url) - Fires when the hash has changed but not the page
Here url is the URL shown in the browser. If you call Router.push(url, as) (or similar), then the value of url will be as.

Here's how to properly listen to the router event routeChangeStart:

const handleRouteChange = url => {
  console.log('App is changing to: ', url)
}

Router.events.on('routeChangeStart', handleRouteChange)
If you no longer want to listen to that event, you can unsubscribe with the off method:

Router.events.off('routeChangeStart', handleRouteChange)
If a route load is cancelled (for example by clicking two links rapidly in succession), routeChangeError will fire. The passed err will contain a cancelled property set to true.

Router.events.on('routeChangeError', (err, url) => {
  if (err.cancelled) {
    console.log(`Route to ${url} was cancelled!`)
  }
})
Shallow Routing
Examples
Shallow routing allows you to change the URL without running getInitialProps. You'll receive the updated pathname and the query via the router prop (injected using withRouter), without losing state.

You can do this by invoking either Router.push or Router.replace with the shallow: true option. Here's an example:

// Current URL is "/"
const href = '/?counter=10'
const as = href
Router.push(href, as, { shallow: true })
Now, the URL is updated to /?counter=10. You can see the updated URL with this.props.router.query inside the Component (make sure you are using withRouter around your Component to inject the router prop).

You can watch for URL changes via componentDidUpdate hook as shown below:

componentDidUpdate(prevProps) {
  const { pathname, query } = this.props.router
  // verify props have changed to avoid an infinite loop
  if (query.id !== prevProps.router.query.id) {
    // fetch data based on the new query
  }
}
NOTES:

Shallow routing works only for same page URL changes. For an example, let's assume we have another page called about, and you run this:

Router.push('/?counter=10', '/about?counter=10', { shallow: true })
Since that's a new page, it'll unload the current page, load the new one and call getInitialProps even though we asked to do shallow routing.

Using a Higher Order Component
Examples
If you want to access the router object inside any component in your app, you can use the withRouter Higher-Order Component. Here's how to use it:

import { withRouter } from 'next/router'

function ActiveLink({ children, router, href }) {
  const style = {
    marginRight: 10,
    color: router.pathname === href ? 'red' : 'black'
  }

  const handleClick = (e) => {
    e.preventDefault()
    router.push(href)
  }

  return (
    <a href={href} onClick={handleClick} style={style}>
      {children}
    </a>
  )
}

export default withRouter(ActiveLink)
The above router object comes with an API similar to next/router.

Prefetching Pages
?? This is a production only feature ??

Examples
Next.js has an API which allows you to prefetch pages.

Since Next.js server-renders your pages, this allows all the future interaction paths of your app to be instant. Effectively Next.js gives you the great initial download performance of a website, with the ahead-of-time download capabilities of an app. Read more.

With prefetching Next.js only downloads JS code. When the page is getting rendered, you may need to wait for the data.

With <Link>
You can add prefetch prop to any <Link> and Next.js will prefetch those pages in the background.

import Link from 'next/link'

function Header() {
  return (
    <nav>
      <ul>
        <li>
          <Link prefetch href="/">
            <a>Home</a>
          </Link>
        </li>
        <li>
          <Link prefetch href="/about">
            <a>About</a>
          </Link>
        </li>
        <li>
          <Link prefetch href="/contact">
            <a>Contact</a>
          </Link>
        </li>
      </ul>
    </nav>
  )
}

export default Header
Imperatively
Most prefetching needs are addressed by <Link />, but we also expose an imperative API for advanced usage:

import { withRouter } from 'next/router'

function MyLink({ router }) {
  return (
    <div>
      <a onClick={() => setTimeout(() => router.push('/dynamic'), 100)}>
        A route transition will happen after 100ms
      </a>
      {// but we can prefetch it!
      router.prefetch('/dynamic')}
    </div>
  )
}

export default withRouter(MyLink)
The router instance should be only used inside the client side of your app though. In order to prevent any error regarding this subject, when rendering the Router on the server side, use the imperatively prefetch method in the componentDidMount() lifecycle method.

import React from 'react'
import { withRouter } from 'next/router'

class MyLink extends React.Component {
  componentDidMount() {
    const { router } = this.props
    router.prefetch('/dynamic')
  }

  render() {
    const { router } = this.props

    return (
       <div>
        <a onClick={() => setTimeout(() => router.push('/dynamic'), 100)}>
          A route transition will happen after 100ms
        </a>
      </div>
    )
  }
}

export default withRouter(MyLink)
Custom server and routing
Examples
Typically you start your next server with next start. It's possible, however, to start a server 100% programmatically in order to customize routes, use route patterns, etc.

When using a custom server with a server file, for example called server.js, make sure you update the scripts key in package.json to:

{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
This example makes /a resolve to ./pages/b, and /b resolve to ./pages/a:

// This file doesn't go through babel or webpack transformation.
// Make sure the syntax and sources this file requires are compatible with the current node version you are running
// See https://github.com/zeit/next.js/issues/1245 for discussions on Universal Webpack or universal Babel
const { createServer } = require('http')
const { parse } = require('url')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    // Be sure to pass `true` as the second argument to `url.parse`.
    // This tells it to parse the query portion of the URL.
    const parsedUrl = parse(req.url, true)
    const { pathname, query } = parsedUrl

    if (pathname === '/a') {
      app.render(req, res, '/b', query)
    } else if (pathname === '/b') {
      app.render(req, res, '/a', query)
    } else {
      handle(req, res, parsedUrl)
    }
  }).listen(3000, err => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
The next API is as follows:

next(opts: object)
Supported options:

dev (bool) whether to launch Next.js in dev mode - default false
dir (string) where the Next project is located - default '.'
quiet (bool) Hide error messages containing server information - default false
conf (object) the same object you would use in next.config.js - default {}
Then, change your start script to NODE_ENV=production node server.js.

Disabling file-system routing
By default, Next will serve each file in /pages under a pathname matching the filename (eg, /pages/some-file.js is served at site.com/some-file.

If your project uses custom routing, this behavior may result in the same content being served from multiple paths, which can present problems with SEO and UX.

To disable this behavior & prevent routing based on files in /pages, simply set the following option in your next.config.js:

// next.config.js
module.exports = {
  useFileSystemPublicRoutes: false
}
Note that useFileSystemPublicRoutes simply disables filename routes from SSR; client-side routing may still access those paths. If using this option, you should guard against navigation to routes you do not want programmatically.

You may also wish to configure the client-side Router to disallow client-side redirects to filename routes; please refer to Intercepting popstate.

Dynamic assetPrefix
Sometimes we need to set the assetPrefix dynamically. This is useful when changing the assetPrefix based on incoming requests. For that, we can use app.setAssetPrefix.

Here's an example usage of it:

const next = require('next')
const http = require('http')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handleNextRequests = app.getRequestHandler()

app.prepare().then(() => {
  const server = new http.Server((req, res) => {
    // Add assetPrefix support based on the hostname
    if (req.headers.host === 'my-app.com') {
      app.setAssetPrefix('http://cdn.com/myapp')
    } else {
      app.setAssetPrefix('')
    }

    handleNextRequests(req, res)
  })

  server.listen(port, (err) => {
    if (err) {
      throw err
    }

    console.log(`> Ready on http://localhost:${port}`)
  })
})
Dynamic Import
Examples
Next.js supports TC39 dynamic import proposal for JavaScript. With that, you could import JavaScript modules (inc. React Components) dynamically and work with them.

You can think dynamic imports as another way to split your code into manageable chunks. Since Next.js supports dynamic imports with SSR, you could do amazing things with it.

Here are a few ways to use dynamic imports.

1. Basic Usage (Also does SSR)
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
2. With Custom Loading Component
import dynamic from 'next/dynamic'

const DynamicComponentWithCustomLoading = dynamic(() => import('../components/hello2'), {
  loading: () => <p>...</p>
})

function Home() {
  return (
    <div>
      <Header />
      <DynamicComponentWithCustomLoading />
      <p>HOME PAGE is here!</p>
    </div>
  )
}

export default Home
3. With No SSR
import dynamic from 'next/dynamic'

const DynamicComponentWithNoSSR = dynamic(() => import('../components/hello3'), {
  ssr: false
})

function Home() {
  return (
    <div>
      <Header />
      <DynamicComponentWithNoSSR />
      <p>HOME PAGE is here!</p>
    </div>
  )
}

export default Home
4. With Multiple Modules At Once
import dynamic from 'next/dynamic'

const HelloBundle = dynamic({
  modules: () => {
    const components = {
      Hello1: () => import('../components/hello1'),
      Hello2: () => import('../components/hello2')
    }

    return components
  },
  render: (props, { Hello1, Hello2 }) =>
    <div>
      <h1>
        {props.title}
      </h1>
      <Hello1 />
      <Hello2 />
    </div>
})

function DynamicBundle() {
  return <HelloBundle title="Dynamic Bundle" />
}

export default DynamicBundle
Custom <App>
Examples
Next.js uses the App component to initialize pages. You can override it and control the page initialization. Which allows you to do amazing things like:

Persisting layout between page changes
Keeping state when navigating pages
Custom error handling using componentDidCatch
Inject additional data into pages (for example by processing GraphQL queries)
To override, create the ./pages/_app.js file and override the App class as shown below:

import React from 'react'
import App, { Container } from 'next/app'

class MyApp extends App {
  static async getInitialProps({ Component, ctx }) {
    let pageProps = {}

    if (Component.getInitialProps) {
      pageProps = await Component.getInitialProps(ctx)
    }

    return { pageProps }
  }

  render () {
    const { Component, pageProps } = this.props

    return (
      <Container>
        <Component {...pageProps} />
      </Container>
    )
  }
}

export default MyApp
Custom <Document>
Examples
Is rendered on the server side
Is used to change the initial server side rendered document markup
Commonly used to implement server side rendering for css-in-js libraries like styled-components or emotion. styled-jsx is included with Next.js by default.
Pages in Next.js skip the definition of the surrounding document's markup. For example, you never include <html>, <body>, etc. To override that default behavior, you must create a file at ./pages/_document.js, where you can extend the Document class:

// _document is only rendered on the server side and not on the client side
// Event handlers like onClick can't be added to this file

// ./pages/_document.js
import Document, { Head, Main, NextScript } from 'next/document'

class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx)
    return { ...initialProps }
  }

  render() {
    return (
      <html>
        <Head>
          <style>{`body { margin: 0 } /* custom! */`}</style>
        </Head>
        <body className="custom_class">
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}

export default MyDocument
All of <Head />, <Main /> and <NextScript /> are required for page to be properly rendered.

Note: React-components outside of <Main /> will not be initialised by the browser. Do not add application logic here. If you need shared components in all your pages (like a menu or a toolbar), take a look at the App component instead.

The ctx object is equivalent to the one received in all getInitialProps hooks, with one addition:

renderPage (Function) a callback that executes the actual React rendering logic (synchronously). It's useful to decorate this function in order to support server-rendering wrappers like Aphrodite's renderStatic
Customizing renderPage
?? It should be noted that the only reason you should be customizing renderPage is for usage with css-in-js libraries that need to wrap the application to properly work with server-rendering. ??

It takes as argument an options object for further customization
import Document from 'next/document'

class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const originalRenderPage = ctx.renderPage

    ctx.renderPage = () => originalRenderPage({
      // useful for wrapping the whole react tree
      enhanceApp: App => App,
      // userful for wrapping in a per-page basis
      enhanceComponent: Component => Component
    })

    // Run the parent `getInitialProps` using `ctx` that now includes our custom `renderPage`
    const initialProps = await Document.getInitialProps(ctx)

    return initialProps
  }
}

export default MyDocument
Custom error handling
404 or 500 errors are handled both client and server side by a default component error.js. If you wish to override it, define a _error.js in the pages folder:

?? The pages/_error.js component is only used in production. In development you get an error with call stack to know where the error originated from. ??

import React from 'react'

class Error extends React.Component {
  static getInitialProps({ res, err }) {
    const statusCode = res ? res.statusCode : err ? err.statusCode : null;
    return { statusCode }
  }

  render() {
    return (
      <p>
        {this.props.statusCode
          ? `An error ${this.props.statusCode} occurred on server`
          : 'An error occurred on client'}
      </p>
    )
  }
}

export default Error
Reusing the built-in error page
If you want to render the built-in error page you can by using next/error:

import React from 'react'
import Error from 'next/error'
import fetch from 'isomorphic-unfetch'

class Page extends React.Component {
  static async getInitialProps() {
    const res = await fetch('https://api.github.com/repos/zeit/next.js')
    const errorCode = res.statusCode > 200 ? res.statusCode : false
    const json = await res.json()

    return { errorCode, stars: json.stargazers_count }
  }

  render() {
    if (this.props.errorCode) {
      return <Error statusCode={this.props.errorCode} />
    }

    return (
      <div>
        Next stars: {this.props.stars}
      </div>
    )
  }
}

export default Page
If you have created a custom error page you have to import your own _error component from ./_error instead of next/error

Custom configuration
For custom advanced behavior of Next.js, you can create a next.config.js in the root of your project directory (next to pages/ and package.json).

Note: next.config.js is a regular Node.js module, not a JSON file. It gets used by the Next server and build phases, and not included in the browser build.

// next.config.js
module.exports = {
  /* config options here */
}
Or use a function:

module.exports = (phase, {defaultConfig}) => {
  return {
    /* config options here */
  }
}
phase is the current context in which the configuration is loaded. You can see all phases here: constants Phases can be imported from next/constants:

const {PHASE_DEVELOPMENT_SERVER} = require('next/constants')
module.exports = (phase, {defaultConfig}) => {
  if(phase === PHASE_DEVELOPMENT_SERVER) {
    return {
      /* development only config options here */
    }
  }

  return {
    /* config options for all phases except development here */
  }
}
Setting a custom build directory
You can specify a name to use for a custom build directory. For example, the following config will create a build folder instead of a .next folder. If no configuration is specified then next will create a .next folder.

// next.config.js
module.exports = {
  distDir: 'build'
}
Disabling etag generation
You can disable etag generation for HTML pages depending on your cache strategy. If no configuration is specified then Next will generate etags for every page.

// next.config.js
module.exports = {
  generateEtags: false
}
Configuring the onDemandEntries
Next exposes some options that give you some control over how the server will dispose or keep in memories pages built:

module.exports = {
  onDemandEntries: {
    // period (in ms) where the server will keep pages in the buffer
    maxInactiveAge: 25 * 1000,
    // number of pages that should be kept simultaneously without being disposed
    pagesBufferLength: 2,
    // optionally configure a port for the onDemandEntries WebSocket, not needed by default
    websocketPort: 3001,
    // optionally configure a proxy path for the onDemandEntries WebSocket, not need by default
    websocketProxyPath: '/hmr',
    // optionally configure a proxy port for the onDemandEntries WebSocket, not need by default
    websocketProxyPort: 7002,
  },
}
This is development-only feature. If you want to cache SSR pages in production, please see SSR-caching example.

Configuring extensions looked for when resolving pages in pages
Aimed at modules like @zeit/next-typescript, that add support for pages ending in .ts. pageExtensions allows you to configure the extensions looked for in the pages directory when resolving pages.

// next.config.js
module.exports = {
  pageExtensions: ['jsx', 'js']
}
Configuring the build ID
Next.js uses a constant generated at build time to identify which version of your application is being served. This can cause problems in multi-server deployments when next build is ran on every server. In order to keep a static build id between builds you can provide the generateBuildId function:

// next.config.js
module.exports = {
  generateBuildId: async () => {
    // For example get the latest git commit hash here
    return 'my-build-id'
  }
}
To fall back to the default of generating a unique id return null from the function:

module.exports = {
  generateBuildId: async () => {
    // When process.env.YOUR_BUILD_ID is undefined we fall back to the default
    if(process.env.YOUR_BUILD_ID) {
      return process.env.YOUR_BUILD_ID
    }

    return null
  }
}
Configuring next process script
You can pass any node arguments to next CLI command.

NODE_OPTIONS="--throw-deprecation" next
NODE_OPTIONS="-r esm" next
--inspect is a special case since it binds to a port and can't double-bind to the child process the next CLI creates.

next start --inspect
Customizing webpack config
Examples
Some commonly asked for features are available as modules:

@zeit/next-css
@zeit/next-sass
@zeit/next-less
@zeit/next-preact
@zeit/next-typescript
Warning: The webpack function is executed twice, once for the server and once for the client. This allows you to distinguish between client and server configuration using the isServer property

Multiple configurations can be combined together with function composition. For example:

const withTypescript = require('@zeit/next-typescript')
const withSass = require('@zeit/next-sass')

module.exports = withTypescript(withSass({
  webpack(config, options) {
    // Further custom configuration here
    return config
  }
}))
In order to extend our usage of webpack, you can define a function that extends its config via next.config.js.

// next.config.js is not transformed by Babel. So you can only use javascript features supported by your version of Node.js.

module.exports = {
  webpack: (config, { buildId, dev, isServer, defaultLoaders }) => {
    // Perform customizations to webpack config
    // Important: return the modified config
    return config
  },
  webpackDevMiddleware: config => {
    // Perform customizations to webpack dev middleware config
    // Important: return the modified config
    return config
  }
}
The second argument to webpack is an object containing properties useful when customizing its configuration:

buildId - String the build id used as a unique identifier between builds
dev - Boolean shows if the compilation is done in development mode
isServer - Boolean shows if the resulting configuration will be used for server side (true), or client size compilation (false).
defaultLoaders - Object Holds loader objects Next.js uses internally, so that you can use them in custom configuration
babel - Object the babel-loader configuration for Next.js.
hotSelfAccept - Object the hot-self-accept-loader configuration. This loader should only be used for advanced use cases. For example @zeit/next-typescript adds it for top-level typescript pages.
Example usage of defaultLoaders.babel:

// Example next.config.js for adding a loader that depends on babel-loader
// This source was taken from the @zeit/next-mdx plugin source:
// https://github.com/zeit/next-plugins/blob/master/packages/next-mdx
module.exports = {
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.mdx/,
      use: [
        options.defaultLoaders.babel,
        {
          loader: '@mdx-js/loader',
          options: pluginOptions.options
        }
      ]
    })

    return config
  }
}
Customizing babel config
Examples
In order to extend our usage of babel, you can simply define a .babelrc file at the root of your app. This file is optional.

If found, we're going to consider it the source of truth, therefore it needs to define what next needs as well, which is the next/babel preset.

This is designed so that you are not surprised by modifications we could make to the babel configurations.

Here's an example .babelrc file:

{
  "presets": ["next/babel"],
  "plugins": []
}
The next/babel preset includes everything needed to transpile React applications. This includes:

preset-env
preset-react
plugin-proposal-class-properties
plugin-proposal-object-rest-spread
plugin-transform-runtime
styled-jsx
These presets / plugins should not be added to your custom .babelrc. Instead, you can configure them on the next/babel preset:

{
  "presets": [
    ["next/babel", {
      "preset-env": {},
      "transform-runtime": {},
      "styled-jsx": {},
      "class-properties": {}
    }]
  ],
  "plugins": []
}
The modules option on "preset-env" should be kept to false otherwise webpack code splitting is disabled.

Exposing configuration to the server / client side
There is a common need in applications to provide configuration values.

Next.js supports 2 ways of providing configuration:

Build-time configuration
Runtime configuration
Build time configuration
The way build-time configuration works is by inlining the provided values into the Javascript bundle.

You can add the env key in next.config.js:

// next.config.js
module.exports = {
  env: {
    customKey: 'value'
  }
}
This will allow you to use process.env.customKey in your code. For example:

// pages/index.js
function Index() {
  return <h1>The value of customEnv is: {process.env.customEnv}</h1>
}

export default Index
Runtime configuration
?? Note that this option is not available when using target: 'serverless'

?? Generally you want to use build-time configuration to provide your configuration. The reason for this is that runtime configuration adds a small rendering / initialization overhead.

The next/config module gives your app access to the publicRuntimeConfig and serverRuntimeConfig stored in your next.config.js.

Place any server-only runtime config under a serverRuntimeConfig property.

Anything accessible to both client and server-side code should be under publicRuntimeConfig.

// next.config.js
module.exports = {
  serverRuntimeConfig: { // Will only be available on the server side
    mySecret: 'secret',
    secondSecret: process.env.SECOND_SECRET // Pass through env variables
  },
  publicRuntimeConfig: { // Will be available on both server and client
    staticFolder: '/static',
  }
}
// pages/index.js
import getConfig from 'next/config'
// Only holds serverRuntimeConfig and publicRuntimeConfig from next.config.js nothing else.
const {serverRuntimeConfig, publicRuntimeConfig} = getConfig()

console.log(serverRuntimeConfig.mySecret) // Will only be available on the server side
console.log(publicRuntimeConfig.staticFolder) // Will be available on both server and client

function MyImage() {
  return (
    <div>
      <img src={`${publicRuntimeConfig.staticFolder}/logo.png`} alt="logo" />
    </div>
  )
}

export default MyImage
Starting the server on alternative hostname
To start the development server using a different default hostname you can use --hostname hostname_here or -H hostname_here option with next dev. This will start a TCP server listening for connections on the provided host.

CDN support with Asset Prefix
To set up a CDN, you can set up the assetPrefix setting and configure your CDN's origin to resolve to the domain that Next.js is hosted on.

const isProd = process.env.NODE_ENV === 'production'
module.exports = {
  // You may only need to add assetPrefix in the production.
  assetPrefix: isProd ? 'https://cdn.mydomain.com' : ''
}
Note: Next.js will automatically use that prefix in the scripts it loads, but this has no effect whatsoever on /static. If you want to serve those assets over the CDN, you'll have to introduce the prefix yourself. One way of introducing a prefix that works inside your components and varies by environment is documented in this example.

If your CDN is on a separate domain and you would like assets to be requested using a CORS aware request you can set a config option for that.

// next.config.js
module.exports = {
  crossOrigin: 'anonymous'
}
Production deployment
To deploy, instead of running next, you want to build for production usage ahead of time. Therefore, building and starting are separate commands:

next build
next start
For example, to deploy with now a package.json like follows is recommended:

{
  "name": "my-app",
  "dependencies": {
    "next": "latest"
  },
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
Then run now and enjoy!

Next.js can be deployed to other hosting solutions too. Please have a look at the 'Deployment' section of the wiki.

Note: NODE_ENV is properly configured by the next subcommands, if absent, to maximize performance. if you�re using Next.js programmatically, it�s your responsibility to set NODE_ENV=production manually!

Note: we recommend putting .next, or your custom dist folder, in .gitignore or .npmignore. Otherwise, use files or now.files to opt-into a whitelist of files you want to deploy, excluding .next or your custom dist folder.

Serverless deployment
Examples
Serverless deployment dramatically improves reliability and scalability by splitting your application into smaller parts (also called lambdas). In the case of Next.js, each page in the pages directory becomes a serverless lambda.

There are a number of benefits to serverless. The referenced link talks about some of them in the context of Express, but the principles apply universally: serverless allows for distributed points of failure, infinite scalability, and is incredibly affordable with a "pay for what you use" model.

To enable serverless mode in Next.js, add the serverless build target in next.config.js:

// next.config.js
module.exports = {
  target: "serverless",
};
The serverless target will output a single lambda per page. This file is completely standalone and doesn't require any dependencies to run:

pages/index.js => .next/serverless/pages/index.js
pages/about.js => .next/serverless/pages/about.js
The signature of the Next.js Serverless function is similar to the Node.js HTTP server callback:

export function render(req: http.IncomingMessage, res: http.ServerResponse) => void
http.IncomingMessage
http.ServerResponse
void refers to the function not having a return value and is equivalent to JavaScript's undefined. Calling the function will finish the request.
One Level Lower
Next.js provides low-level APIs for serverless deployments as hosting platforms have different function signatures. In general you will want to wrap the output of a Next.js serverless build with a compatability layer.

For example if the platform supports the Node.js http.Server class:

const http = require("http");
const page = require("./.next/serverless/pages/about.js");
const server = new http.Server((req, res) => page.render(req, res));
server.listen(3000, () => console.log("Listening on http://localhost:3000"));
For specific platform examples see the examples section above.

Summary
Low-level API for implementing serverless deployment
Every page in the pages directory becomes a serverless function (lambda)
Creates the smallest possible serverless function (50Kb base zip size)
Optimized for fast cold start of the function
The serverless function has 0 dependencies (they are included in the function bundle)
Uses the http.IncomingMessage and http.ServerResponse from Node.js
opt-in using target: 'serverless' in next.config.js
Does not load next.config.js when executing the function, note that this means publicRuntimeConfig / serverRuntimeConfig are not supported
Browser support
Next.js supports IE11 and all modern browsers out of the box using @babel/preset-env. In order to support IE11 Next.js adds a global Promise polyfill. In cases where your own code or any external NPM dependencies you are using requires features not supported by your target browsers you will need to implement polyfills.

The polyfills example demonstrates the recommended approach to implement polyfills.

Static HTML export
Examples
next export is a way to run your Next.js app as a standalone static app without the need for a Node.js server. The exported app supports almost every feature of Next.js, including dynamic urls, prefetching, preloading and dynamic imports.

The way next export works is by pre-rendering all pages possible to HTML. It does so based on a mapping of pathname key to page object. This mapping is called the exportPathMap.

The page object has 2 values:

page - String the page inside the pages directory to render
query - Object the query object passed to getInitialProps when pre-rendering. Defaults to {}
Usage
Simply develop your app as you normally do with Next.js. Then run:

next build
next export
By default next export doesn't require any configuration. It will generate a default exportPathMap containing the routes to pages inside the pages directory. This default mapping is available as defaultPathMap in the example below.

If your application has dynamic routes you can add a dynamic exportPathMap in next.config.js. This function is asynchronous and gets the default exportPathMap as a parameter.

// next.config.js
module.exports = {
  exportPathMap: async function (defaultPathMap) {
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/readme.md': { page: '/readme' },
      '/p/hello-nextjs': { page: '/post', query: { title: 'hello-nextjs' } },
      '/p/learn-nextjs': { page: '/post', query: { title: 'learn-nextjs' } },
      '/p/deploy-nextjs': { page: '/post', query: { title: 'deploy-nextjs' } }
    }
  }
}
Note that if the path ends with a directory, it will be exported as /dir-name/index.html, but if it ends with an extension, it will be exported as the specified filename, e.g. /readme.md above. If you use a file extension other than .html, you may need to set the Content-Type header to text/html when serving this content.

Then simply run these commands:

next build
next export
For that you may need to add a NPM script to package.json like this:

{
  "scripts": {
    "build": "next build",
    "export": "npm run build && next export"
  }
}
And run it at once with:

npm run export
Then you have a static version of your app in the out directory.

You can also customize the output directory. For that run next export -h for the help.

Now you can deploy the out directory to any static hosting service. Note that there is an additional step for deploying to GitHub Pages, documented here.

For an example, simply visit the out directory and run following command to deploy your app to ZEIT Now.

now
Copying custom files
In case you have to copy custom files like a robots.txt or generate a sitemap.xml you can do this inside of exportPathMap. exportPathMap gets a few contextual parameter to aid you with creating/copying files:

dev - true when exportPathMap is being called in development. false when running next export. In development exportPathMap is used to define routes and behavior like copying files is not required.
dir - Absolute path to the project directory
outDir - Absolute path to the out directory (configurable with -o or --outdir). When dev is true the value of outDir will be null.
distDir - Absolute path to the .next directory (configurable using the distDir config key)
buildId - The buildId the export is running for
// next.config.js
const fs = require('fs')
const { join } = require('path')
const { promisify } = require('util')
const copyFile = promisify(fs.copyFile)

module.exports = {
  exportPathMap: async function (defaultPathMap, {dev, dir, outDir, distDir, buildId}) {
    if (dev) {
      return defaultPathMap
    }
    // This will copy robots.txt from your project root into the out directory
    await copyFile(join(dir, 'robots.txt'), join(outDir, 'robots.txt'))
    return defaultPathMap
  }
}
Limitation
With next export, we build a HTML version of your app. At export time we will run getInitialProps of your pages.

The req and res fields of the context object passed to getInitialProps are not available as there is no server running.

You won't be able to render HTML dynamically when static exporting, as we pre-build the HTML files. If you want to do dynamic rendering use next start or the custom server API

Multi Zones
Examples
A zone is a single deployment of a Next.js app. Just like that, you can have multiple zones. Then you can merge them as a single app.

For an example, you can have two zones like this:

https://docs.my-app.com for serving /docs/**
https://ui.my-app.com for serving all other pages
With multi zones support, you can merge both these apps into a single one. Which allows your customers to browse it using a single URL. But you can develop and deploy both apps independently.

This is exactly the same concept as microservices, but for frontend apps.

How to define a zone
There are no special zones related APIs. You only need to do following things:

Make sure to keep only the pages you need in your app. (For an example, https://ui.my-app.com should not contain pages for /docs/**)
Make sure your app has an assetPrefix. (You can also define the assetPrefix dynamically.)
How to merge them
You can merge zones using any HTTP proxy.

You can use micro proxy as your local proxy server. It allows you to easily define routing rules like below:

{
  "rules": [
    {"pathname": "/docs**", "method":["GET", "POST", "OPTIONS"], "dest": "https://docs.my-app.com"},
    {"pathname": "/**", "dest": "https://ui.my-app.com"}
  ]
}
For the production deployment, you can use the path alias feature if you are using ZEIT now. Otherwise, you can configure your existing proxy server to route HTML pages using a set of rules as shown above.

Recipes
Setting up 301 redirects
Dealing with SSR and server only modules
Building with React-Material-UI-Next-Express-Mongoose-Mongodb
Build a SaaS Product with React-Material-UI-Next-MobX-Express-Mongoose-MongoDB-TypeScript
FAQ
Is this production ready?
How big is it?
Is this like `create-react-app`?
How do I use CSS-in-JS solutions?
What syntactic features are transpiled? How do I change them?
Why a new Router?
How do I define a custom fancy route?
How do I fetch data?
Can I use it with GraphQL?
Can I use it with Redux?
Can I use Next with my favorite Javascript library or toolkit?
What is this inspired by?
Contributing
Please see our contributing.md

Authors
Arunoda Susiripala (@arunoda) � ZEIT
Tim Neutkens (@timneutkens) � ZEIT
Naoyuki Kanezawa (@nkzawa) � ZEIT
Tony Kovanen (@tonykovanen) � ZEIT
Guillermo Rauch (@rauchg) � ZEIT
Dan Zajdband (@impronunciable) � Knight-Mozilla / Coral Project