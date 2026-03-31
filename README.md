# kernel browse

browser infrastructure for AI agents and web automations. cloud browsers in <30ms, stealth mode, managed auth, and playwright execution — right from cursor.

trusted by 3,000+ teams.

## what's included

### mcp server
connects cursor to kernel's cloud browser platform. browsers spin up in the cloud in <30ms — no local chrome needed.

available tools:
- **create_browser** — launch cloud browsers with stealth mode, proxies, custom viewports, and profiles
- **execute_playwright_code** — run playwright/typescript against any browser session
- **take_screenshot** — capture browser screenshots
- **setup_profile** — save and reuse browser sessions with persistent cookies and logins
- **search_docs** — search kernel documentation

### skills
- **browser-automation** — patterns for building browser automations with the kernel sdk and playwright
- **kernel-mcp** — guide for using kernel's mcp tools effectively
- **web-scraping** — scraping patterns with stealth mode and proxies
- **kernel-typescript-sdk** — typescript sdk patterns: playwright, CDP connections, stagehand integration, deployable apps
- **kernel-python-sdk** — python sdk patterns: playwright, browser-use integration, async context managers, deployable apps
- **managed-auth** — authenticate AI agents across websites with persistent login sessions

### rules
- **kernel-best-practices** — always clean up browsers, use stealth for bot detection, handle errors

## setup

1. install from the cursor marketplace
2. authenticate with your kernel account (oauth — opens in browser)
3. start using kernel tools in cursor agent mode

no api key needed for setup — oauth handles auth automatically. for CI/CD or headless environments, use an api key:

```json
{
  "mcpServers": {
    "kernel": {
      "url": "https://mcp.onkernel.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_KERNEL_API_KEY"
      }
    }
  }
}
```

## get started

- sign up at [kernel.sh](https://kernel.sh)
- read the [docs](https://kernel.sh/docs)
- check out the [typescript sdk](https://github.com/onkernel/kernel-typescript-sdk) and [python sdk](https://github.com/onkernel/kernel-python-sdk)
- open source browser image: [kernel-images](https://github.com/onkernel/kernel-images)

## license

mit
