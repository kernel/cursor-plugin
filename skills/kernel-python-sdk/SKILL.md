---
name: kernel-python-sdk
description: build browser automation scripts using the kernel python sdk with playwright and remote browser management. use when writing python code that creates browsers, runs playwright, or automates websites with kernel.
---

# kernel python sdk

install: `pip install kernel`

## core architecture

same namespaces as typescript: `kernel.browsers`, `kernel.playwright`, `kernel.computer`, `kernel.pools`, `kernel.profiles`, `kernel.auth`, `kernel.proxies`, `kernel.extensions`, `kernel.deployments`, `kernel.invocations`.

## two automation approaches

### 1. CDP connection (complex automations)

```python
from kernel import Kernel
from playwright.async_api import async_playwright

kernel = Kernel()
browser = kernel.browsers.create(stealth=True, timeout_seconds=300)

async with async_playwright() as pw:
    browser_ctx = await pw.chromium.connect_over_cdp(browser.cdp_ws_url)
    page = browser_ctx.contexts[0].pages[0]

    try:
        await page.goto("https://example.com")
        title = await page.title()
        print(title)
    finally:
        await browser_ctx.close()
        kernel.browsers.delete_by_id(browser.session_id)
```

### 2. server-side execution (simple scripts)

```python
result = kernel.playwright.execute(
    browser.session_id,
    """
    await page.goto("https://example.com")
    return await page.title()
    """
)
```

## browser-use integration

connect browser-use AI agent to kernel:

```python
from kernel import Kernel
from browser_use import Agent, Browser, BrowserConfig

kernel = Kernel()
browser = kernel.browsers.create(stealth=True)

config = BrowserConfig(cdp_url=browser.cdp_ws_url)
agent_browser = Browser(config=config)

agent = Agent(
    task="Find the top story on Hacker News",
    llm=your_llm,
    browser=agent_browser,
)

result = await agent.run()
kernel.browsers.delete_by_id(browser.session_id)
```

## action handler pattern (deployable apps)

```python
from kernel import Kernel

kernel = Kernel()
app = kernel.app("my-app")

@app.action("get-page-title")
async def get_page_title(ctx, payload):
    browser = await ctx.browsers.create(stealth=True)
    # ... automate and return result
    return {"title": "Example"}
```

deploy with `kernel deploy main.py`.

## managed auth

```python
# create an auth connection
connection = kernel.auth.connections.create(
    domain="github.com",
    label="my-github"
)

# get a login URL for the user
login = kernel.auth.connections.login(connection.id)
print(login.url)  # user completes login here

# use the authenticated session
browser = kernel.browsers.create(
    auth_connection_id=connection.id
)
# browser starts already logged in
```

## async context manager pattern

```python
from dataclasses import dataclass

@dataclass
class KernelBrowserSession:
    kernel: Kernel
    stealth: bool = True
    timeout_seconds: int = 300

    async def __aenter__(self):
        self.browser = self.kernel.browsers.create(
            stealth=self.stealth,
            timeout_seconds=self.timeout_seconds,
        )
        self.pw = await async_playwright().start()
        self.browser_ctx = await self.pw.chromium.connect_over_cdp(self.browser.cdp_ws_url)
        self.page = self.browser_ctx.contexts[0].pages[0]
        return self

    async def __aexit__(self, *args):
        await self.browser_ctx.close()
        await self.pw.stop()
        self.kernel.browsers.delete_by_id(self.browser.session_id)

# usage
async with KernelBrowserSession(kernel) as session:
    await session.page.goto("https://example.com")
```
