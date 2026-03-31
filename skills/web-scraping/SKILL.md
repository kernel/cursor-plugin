---
name: web-scraping
description: scrape websites, extract data, crawl pages, and browse the web using kernel cloud browsers with stealth mode, proxies, and playwright.
---

# web scraping with kernel

kernel gives you real chrome instances in the cloud for scraping sites that block traditional HTTP scrapers. browsers spin up in <30ms, run in isolated VMs with full javascript rendering, and sessions can last up to 72 hours.

## basic scraping pattern

```typescript
import Kernel from "@onkernel/sdk";
import { chromium } from "playwright";

const kernel = new Kernel();
const browser = await kernel.browsers.create({
  stealth: true,         // adds recaptcha solver + residential proxy
  timeout_seconds: 120,
});

const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
const page = pw.contexts()[0].pages()[0];

try {
  await page.goto("https://example.com/products", { waitUntil: "networkidle" });

  const products = await page.$$eval(".product-card", cards =>
    cards.map(card => ({
      name: card.querySelector("h2")?.textContent?.trim(),
      price: card.querySelector(".price")?.textContent?.trim(),
      url: card.querySelector("a")?.href,
    }))
  );

  return products;
} finally {
  await pw.close();
  await kernel.browsers.deleteByID(browser.session_id);
}
```

## stealth mode

stealth mode automatically adds a recaptcha solver and residential proxy to your browsers. we solve captchas and manage proxies to help you see fewer of them.

always use `stealth: true` for scraping. it handles:
- navigator property spoofing
- webgl fingerprint randomization
- automatic captcha solving
- residential proxy routing

```typescript
const browser = await kernel.browsers.create({ stealth: true });
```

## proxies

for sites with IP-based blocking, add a dedicated proxy. quality ranking for bot detection avoidance:

1. **mobile** — highest quality, mobile IP ranges
2. **residential** — home ISP IPs, very hard to detect
3. **ISP** — datacenter IPs registered to ISPs
4. **datacenter** — cheapest, most likely to be blocked

```typescript
const browser = await kernel.browsers.create({
  stealth: true,
  proxy_id: "your-proxy-id",
});
```

stealth mode already includes a residential proxy. add a dedicated proxy only if you need geo-targeting or the default proxy isn't sufficient.

## handling dynamic content

### wait for elements

```typescript
await page.waitForSelector(".product-list .item", { timeout: 10000 });
await page.goto(url, { waitUntil: "networkidle" });

// wait for a specific API response
const response = await page.waitForResponse(resp =>
  resp.url().includes("/api/products") && resp.status() === 200
);
const data = await response.json();
```

### infinite scroll

```typescript
async function scrollToBottom(page) {
  let previousHeight = 0;
  while (true) {
    const currentHeight = await page.evaluate(() => document.body.scrollHeight);
    if (currentHeight === previousHeight) break;
    previousHeight = currentHeight;
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await page.waitForTimeout(1500);
  }
}
```

### pagination

```typescript
const allItems = [];
let pageNum = 1;

while (true) {
  await page.goto(`https://example.com/items?page=${pageNum}`, {
    waitUntil: "networkidle",
  });

  const items = await page.$$eval(".item", els =>
    els.map(el => el.textContent?.trim())
  );

  if (items.length === 0) break;
  allItems.push(...items);
  pageNum++;
  await page.waitForTimeout(1000 + Math.random() * 1000);
}
```

## scaling with concurrent browsers

spin up multiple browsers in parallel. kernel handles the infra — you just create and connect.

```typescript
const urls = ["https://example.com/1", "https://example.com/2", "https://example.com/3"];

const results = await Promise.all(
  urls.map(async url => {
    const browser = await kernel.browsers.create({
      stealth: true,
      timeout_seconds: 60,
    });
    const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
    const page = pw.contexts()[0].pages()[0];

    try {
      await page.goto(url, { waitUntil: "networkidle" });
      return await page.title();
    } finally {
      await pw.close();
      await kernel.browsers.deleteByID(browser.session_id);
    }
  })
);
```

for high-volume scraping, use browser pools to pre-warm browsers and cut startup latency even further.

## extracting structured data

### tables

```typescript
const tableData = await page.$$eval("table tbody tr", rows =>
  rows.map(row => {
    const cells = row.querySelectorAll("td");
    return Array.from(cells).map(cell => cell.textContent?.trim());
  })
);
```

### JSON-LD / structured data

```typescript
const structuredData = await page.$$eval(
  'script[type="application/ld+json"]',
  scripts => scripts.map(s => JSON.parse(s.textContent || "{}"))
);
```

## respectful scraping

- add delays between requests
- respect `robots.txt`
- set reasonable concurrency limits
- kernel doesn't charge for idle time, but clean up browsers when done
