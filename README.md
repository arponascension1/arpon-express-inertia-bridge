# arpon-express-inertia-bridge

**The Express + Inertia.js bridge middleware for building modern full-stack web applications with React or Vue**

Connect your Express.js backend to Inertia.js frontend. Build single-page applications (SPAs) with server-side rendering support, supporting both React and Vue 3 with Vite bundling.

### What is Inertia.js?

Inertia.js is a framework for building modern single-page applications (SPAs) using classic server-side routing and controllers. This Express middleware bridges the gap between your server and client, eliminating the need for a separate API.

**Perfect for:**
- Full-stack web applications with Node.js + Express
- Server-side rendered (SSR) React or Vue applications
- Building SPAs without managing a separate REST/GraphQL API
- Developers who want server routing with client-side interactivity

## What you need

**Node.js:** 18 or higher  
**Express:** 4.18+ (works with Express 5 too)  
**Frontend:** React (with `@inertiajs/react`) or Vue (with `@inertiajs/vue3`)  
**Build tool:** Vite (for modern, fast bundling)

## Installation

Choose your Express + Inertia stack and run one command:

### React + Express + Vite Setup

For building Express Inertia applications with React and Vite:

```bash
npm i arpon-express-inertia-bridge express ejs @inertiajs/react react react-dom
npm i -D vite @vitejs/plugin-react
```

### Vue 3 + Express + Vite Setup

For building Express Inertia applications with Vue 3 and Vite:

```bash
npm i arpon-express-inertia-bridge express ejs @inertiajs/vue3 vue
npm i -D vite @vitejs/plugin-vue
```

## Quick Start

### Step 1: Set up Express

Create your server file (e.g., `server.js`):

```js
import express from "express";
import path from "node:path";
import inertiaExpress from "arpon-express-inertia-bridge";

const app = express();
const isDev = process.env.NODE_ENV !== "production";

app.set("view engine", "ejs");
app.set("views", path.join(process.cwd(), "views"));

app.use(
  inertiaExpress({
    vite: {
      isDev,
      entry: "resources/js/app.jsx" // use app.js for Vue
    }
  })
);

// Example: render a page with data
app.get("/", async (_req, res) => {
  await res.inertia("Home", { message: "Hello from Inertia + Express" });
});

app.listen(3000, () => console.log("Server running on http://localhost:3000"));
```

### Step 2: Create root template

Create `views/base.ejs` - this is the HTML wrapper for your app:

```ejs
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <%- viteReactRefresh %>
  <x-inertia::head>
    <title>My App</title>
    <meta name="description" content="My App Description" />
  </x-inertia::head>
  <%- viteTags %>
</head>
<body>
  <x-inertia::app />
</body>
</html>
```

### Step 3: Create frontend app

**For React** (`resources/js/app.jsx`):

```jsx
import { createInertiaApp } from "@inertiajs/react";
import { createRoot } from "react-dom/client";

createInertiaApp({
  resolve: (name) => {
    const pages = import.meta.glob("./Pages/**/*.jsx", { eager: true });
    return pages[`./Pages/${name}.jsx`];
  },
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />);
  }
});
```

**For Vue** (`resources/js/app.js`):

```js
import { createApp, h } from "vue";
import { createInertiaApp } from "@inertiajs/vue3";

createInertiaApp({
  resolve: (name) => {
    const pages = import.meta.glob("./Pages/**/*.vue", { eager: true });
    return pages[`./Pages/${name}.vue`];
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) }).use(plugin).mount(el);
  }
});
```

### Step 4: Configure Vite

**For React** (`vite.config.js`):

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: { cors: true },
  build: {
    manifest: true,
    rollupOptions: {
      input: "resources/js/app.jsx"
    }
  }
});
```

**For Vue** (`vite.config.js`):

```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [vue()],
  server: { cors: true },
  build: {
    manifest: true,
    rollupOptions: {
      input: "resources/js/app.js"
    }
  }
});
```

## Core API

### `inertiaExpress(options)` - Express + Inertia Middleware Configuration

Complete configuration options for the Express Inertia bridge middleware:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `rootId` | `string` | `"app"` | Inertia root element id in HTML |
| `version` | `string \| () => Promise<string>` | `undefined` | Asset versioning for cache busting with `X-Inertia-Version` |
| `sharedProps` | `object \| (ctx) => object` | `undefined` | Data shared with every page from Express backend |
| `head` | `string \| string[] \| (ctx) => ...` | `undefined` | Manual head tags resolver |
| `headStrategy` | `"auto" \| "head" \| "headFromPage" \| (ctx) => ...` | `"auto"` | How to handle head tags (SEO meta tags, title, etc.) |
| `title` | `string \| (ctx) => string` | `undefined` | Final title override/formatter for pages |
| `headFromPage` | `boolean \| { pagesPath?: string }` | `true` | Read head data from React/Vue page components |
| `vite.adapter` | `"react" \| "vue"` | `"react"` | Inertia adapter (React or Vue 3) |
| `vite.isDev` | `boolean` | auto-detected | Force dev or production mode |
| `vite.devServerUrl` | `string` | `http://localhost:5173` | Vite dev server URL for development |
| `vite.entry` | `string` | `resources/js/app.jsx` (react) / `resources/js/app.js` (vue) | Client entry file path |
| `vite.devStyleEntry` | `string` | `undefined` | Extra stylesheet for development |
| `vite.manifestPath` | `string` | auto-detected | Path to Vite manifest.json for production |
| `vite.assetsBase` | `string` | `"/"` | Prefix for asset paths |
| `ssr.url` | `string` | `http://127.0.0.1:13714` | URL for server-side rendering service |
| `rootView` | `string` | `"base"` | Express view name (EJS template) for initial page load |
| `viewData` | `object \| (ctx) => object` | `undefined` | Extra data for Express templates |
| `templatePath` | `string` | `undefined` | Custom HTML or EJS template file path |
| `template` | `(ctx) => string` | `undefined` | Custom template renderer function |
| `diagnostics` | `boolean` | `false` | Log warnings for common Inertia + Express setup mistakes |

### Response methods - Express Inertia helpers

Methods added to Express `response` object for Inertia rendering:

- `res.inertia(component, props?)` - Render an Inertia page component with React or Vue
- `res.share(props)` - Share data with all pages across your Express app
- `res.inertiaShare(props)` - Alias of `res.share()`
- `res.inertiaLocation(url)` - Send Inertia redirect response (409 + `X-Inertia-Location` header)

### `validateInertiaExpressOptions(options)`

Utility function that validates your Express Inertia configuration and returns a list of diagnostics for common mistakes:
- Missing root view templates
- Invalid Vite configuration
- Missing entry files
- SSR URL issues
- Mismatched head settings

---

## Template and head behavior

### Express EJS template tags

- `<x-inertia::app />` - Renders the Inertia SPA root element
- `<x-inertia::head>...</x-inertia::head>` - Merges default and page-specific head tags (Laravel-style)
- `<title inertia>` - Automatically normalized for Inertia compatibility

### EJS locals provided by Express Inertia middleware

- `inertia`, `page`, `rootId` - Core Inertia page data
- `inertiaHead`, `head`, `ssrHead` - Head tag data
- `ssrBody`, `inertiaBody` - HTML body content
- `viteReactRefresh`, `viteTags`, `viteCombinedTags`, `inertiaViteTags` - Vite script tags
- Plus any custom data from `viewData` option

---

## Comparison: Express Inertia vs alternatives

| Feature | Express + Inertia | Next.js | Nuxt | Traditional REST API |
| --- | --- | --- | --- | --- |
| **Server routing** | ✅ Express routes | ✅ File-based | ✅ File-based | ❌ Requires API |
| **Full-stack** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ Separate frontend |
| **React support** | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| **Vue support** | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **Vite support** | ✅ Yes | ❌ Webpack/Turbopack | ✅ Yes | ✅ Yes |
| **SSR** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| **Flexibility** | ✅ Use any Express stack | Limited | Limited | ✅ High |

---

## Common use cases

### Building a React Inertia app with Express
Use this package to connect your Express backend to React frontend without building a separate REST API.

### Building a Vue 3 Inertia app with Express
Server-rendered Vue 3 applications with Express routing and Vite bundling.

### Full-stack TypeScript applications
Write type-safe backend and frontend code that works together seamlessly.

### Server-side rendered (SSR) SPAs
Pre-render pages on the server for better performance and SEO in Express Inertia apps.

---

## How it works

1. **Express receives a request** - Your server handles the HTTP request
2. **You call `res.inertia()`** - Render a React or Vue component with server-side data
3. **Middleware injects data into template** - Component name and props are embedded in HTML
4. **Browser renders the page** - React or Vue renders the component client-side
5. **Inertia handles navigation** - Further page transitions are client-side (no full page reloads)

This is a **server-driven SPA** approach - unlike traditional React/Vue apps where the client bootstraps from scratch, Inertia provides the initial page instantly with server-rendered HTML.

## Benefits of Express + Inertia

✅ **Full-stack development** - Write backend and frontend in one codebase  
✅ **Server-side routing** - No need for a separate API layer  
✅ **Modern frontend** - Use React or Vue with all modern tooling  
✅ **Type-safe** - Full TypeScript support  
✅ **SSR support** - Server-side rendering for better performance and SEO  
✅ **Shared data** - Send data from server to all pages automatically  
✅ **Fast dev experience** - Vite for instant HMR (hot module replacement)

## Optional: Server-Side Rendering (SSR)

For even better performance and SEO, enable server-side rendering:

```js
inertiaExpress({
  ssr: { url: "http://127.0.0.1:13714" }
})
```

When configured, the middleware will pre-render HTML on the server before sending to the browser. If SSR fails, it automatically falls back to client-side rendering.

## License

MIT
