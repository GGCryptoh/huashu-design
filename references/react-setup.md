# React + Babel Project Spec

The technical rules you must follow when prototyping with HTML + React + Babel. Break them and it explodes.

## Pinned Script Tags (Use Exactly These)

Put these three script tags in the HTML `<head>` with **fixed versions + integrity hashes**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Don't** use unpinned versions like `react@18` or `react@latest` — you'll hit version drift / cache issues.

**Don't** drop the `integrity` attribute — if a CDN gets hijacked or tampered with, this is the line of defense.

## File Structure

```
project-name/
├── index.html               # main HTML
├── components.jsx           # component file (loaded with type="text/babel")
├── data.js                  # data file
└── styles.css               # extra CSS (optional)
```

Loading order in HTML:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- Then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- Finally the entry point -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Don't** use `type="module"` — it conflicts with Babel.

## Three Inviolable Rules

### Rule 1: `styles` objects must use unique names

**Wrong** (will explode with multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← same name, gets overwritten!
const styles = { container: {...}, header: {...} };
```

**Right**: each component file's `styles` gets a unique prefix.

```jsx
// terminal.jsx
const terminalStyles = { 
  screen: {...}, 
  line: {...} 
};

// sidebar.jsx
const sidebarStyles = { 
  container: {...}, 
  item: {...} 
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

This rule is **non-negotiable**. Every `const styles = {...}` must be replaced with a specific name; otherwise loading multiple components blows up the whole stack.

### Rule 2: Scope is not shared — export to window manually

**Key insight**: each `<script type="text/babel">` is compiled by Babel independently — **scopes don't cross**. The `Terminal` component you defined in `components.jsx` is **undefined by default** in `pages.jsx`.

**Solution**: at the end of each component file, export the components / utilities you want to share to `window`:

```jsx
// end of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // list everything you'll use elsewhere here
});
```

Now `pages.jsx` can use `<Terminal />` directly — JSX will look up `window.Terminal`.

### Rule 3: Don't use `scrollIntoView`

`scrollIntoView` pushes the entire HTML container upward and breaks the web harness's layout. **Never use it.**

Alternatives:
```js
// scroll to a position inside a container
container.scrollTop = targetElement.offsetTop;

// or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling the Claude API (in HTML)

Some native design-agent environments (like Claude.ai Artifacts) ship with a no-config `window.claude.complete`, but most agent environments (Claude Code / Codex / Cursor / Trae / etc.) **do not** have it locally.

If your HTML prototype needs to call an LLM for the demo (e.g., a chat interface), two options:

### Option A: don't call for real — use a mock

Recommended for demos. Write a fake helper that returns a preset response:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // simulate latency
    return "This is a mock response. Replace with the real API for production.";
  }
};
```

### Option B: actually call the Anthropic API

Requires an API key — the user must paste their own key into the HTML to run it. **Never hardcode the key in HTML.**

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: calling the Anthropic API directly from a browser hits CORS issues. If the user's preview environment doesn't support a CORS bypass, this path is dead. Use Option A's mock then, or tell the user a proxy backend is needed.

### Option C: use the agent-side LLM to generate mock data

For a local demo, you can briefly invoke the LLM in the current agent session (or whatever multi-model skill the user has) to generate mock response data and hardcode it into the HTML. The HTML then has zero API dependency at runtime.

## Standard HTML Starter Template

Copy this template as the skeleton for a React prototype:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body { 
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- Entry -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ You defined `const styles` in one file and another file overwrote it. Rename each one specifically.

**`Terminal is not defined`**
→ Cross-file reference; scope doesn't cross. At the end of the file defining `Terminal`, add `Object.assign(window, {Terminal})`.

**Whole page is blank, console shows no errors**
→ Most likely a JSX syntax error that Babel didn't surface to the console. Temporarily swap `babel.min.js` for the unminified `babel.js` for clearer error messages.

**`ReactDOM.createRoot is not a function`**
→ Wrong version. Verify you're on react-dom@18.3.1 (not 17 or other).

**`Objects are not valid as a React child`**
→ You rendered an object instead of JSX / string. Usually `{someObj}` should be `{someObj.name}`.

## How to Split Files in a Big Project

A **single file >1000 lines** is hard to maintain. Splitting strategy:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # base elements: Button, Card, Badge...
│   ├── components.jsx      # business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # home
│   │   ├── detail.jsx      # detail
│   │   └── settings.jsx    # settings
│   ├── router.jsx          # simple router (React state switching)
│   └── app.jsx             # entry component
└── data.js                 # mock data
```

Load in order in HTML:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

**At the end of every file**, `Object.assign(window, {...})` to export what should be shared.
