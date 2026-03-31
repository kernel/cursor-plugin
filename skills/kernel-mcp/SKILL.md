---
name: kernel-mcp
description: manage cloud browsers, take screenshots, run playwright scripts, and manage browser profiles using kernel's MCP tools. use when the kernel MCP server is connected.
---

# kernel MCP tools

when the kernel MCP server is connected, you have direct access to cloud browser management. create, automate, and manage browsers without writing SDK code.

## tools

### browser management
- **create_browser** — launch a cloud browser. returns session ID, CDP websocket URL, and live view URL. supports stealth, proxies, profiles, viewports, headless mode.
- **get_browser** / **list_browsers** — inspect browser sessions
- **delete_browser** — terminate a browser and free resources

### automation
- **execute_playwright_code** — run playwright/typescript against a browser. `page` object is pre-configured. return values come back as the result.
- **take_screenshot** — capture the current browser state

### computer use
- **computer_action** — execute mouse, keyboard, and screenshot actions on a browser session. supports click, move, type, press keys, scroll, drag, screenshots. actions can be batched for lower latency. always include a screenshot as the last action to see the result.

### shell access
- **exec_command** — run shell commands inside a browser VM. returns stdout, stderr, exit code. use for reading files, checking DNS, testing connectivity, running custom scripts inside the browser environment.

### profiles
- **setup_profile** — create or update a browser profile with a guided live session
- **list_profiles** / **delete_profile** — manage saved profiles

### browser pools
- **manage_browser_pools** — manage pools of pre-warmed browser instances for fast acquisition. create pools with a target size, acquire browsers instantly from the pool, release them back when done.

### proxies
- **manage_proxies** — create and manage proxy configurations. supports datacenter, ISP, residential, mobile, and custom proxies with geo-targeting (country, state, city).

### extensions
- **manage_extensions** — list and manage uploaded browser extensions.

### apps
- **list_apps** — list deployed kernel apps
- **invoke_action** — execute an action on a deployed app
- **get_deployment** / **list_deployments** — check deployment status
- **get_invocation** — check action results

### docs
- **search_docs** — search kernel documentation

## prompts

- **kernel-concepts** — explains kernel platform concepts. use when you need background on how kernel works.
- **debug-browser-session** — helps debug browser session issues. use when a session is failing or behaving unexpectedly.

## common workflows

### browse a website
1. `create_browser` with `stealth: true` if the site has bot detection
2. `execute_playwright_code` to navigate and interact
3. `take_screenshot` to verify
4. `delete_browser` to clean up

### execute playwright code

the `page` object is already in scope:

```typescript
await page.goto("https://news.ycombinator.com");
const titles = await page.$$eval(".titleline > a", els =>
  els.slice(0, 10).map(e => e.textContent)
);
return titles;
```

### computer use (CUA)

use `computer_action` for OS-level browser control — mouse, keyboard, screenshots:

```
1. computer_action: screenshot to see the current state
2. computer_action: click_mouse at coordinates to interact
3. computer_action: type_text to enter text
4. computer_action: screenshot to verify
```

batch multiple actions in a single call for lower latency. always end with a screenshot.

### use profiles for persistent sessions
1. `setup_profile` with a name — opens a live browser for you to log in manually
2. `create_browser` with `profile_name` to reuse the session
3. the browser starts already logged in

### scrape with stealth
1. `create_browser` with `stealth: true` — automatically adds recaptcha solver + residential proxy
2. `execute_playwright_code` to extract data
3. `delete_browser` when done

### use browser pools for high throughput
1. `manage_browser_pools` action=create to set up a pool with target size
2. `manage_browser_pools` action=acquire to get a browser instantly
3. use the browser for your task
4. `manage_browser_pools` action=release to return it to the pool

### set up proxies
1. `manage_proxies` action=create with type (residential, ISP, etc.) and geo-targeting
2. `create_browser` with the proxy_id
3. browser traffic routes through the proxy

## tips
- always call `delete_browser` when done — don't leave browsers running
- use `stealth: true` for any site with bot detection
- set `timeout_seconds` as a safety net (default is 60s, max is 72h)
- use `headless: true` for faster execution when you don't need live view
- profiles save cookies and localStorage — use them to avoid re-authenticating
- no charges for idle time — you only pay when browsers are doing work
- use `computer_action` for visual/coordinate-based automation, `execute_playwright_code` for DOM-based automation
- use `exec_command` to debug browser VM issues (check logs, DNS, connectivity)
