---
name: web-scraping
description: scrape websites, extract data, crawl pages, and browse the web at scale using kernel cloud browsers with stealth mode and proxies.
---

# web scraping with kernel

kernel cloud browsers let you scrape at scale with stealth mode, residential proxies, and up to 72-hour sessions. no charges for idle time.

## stealth mode

stealth mode automatically adds a recaptcha solver and residential proxy to your browsers. use it for any site with bot detection.

```typescript
const browser = await kernel.browsers.create({ stealth: true });
```

## proxies

proxy quality for bot detection avoidance, best to worst:
- **mobile** — highest quality, most expensive
- **residential** — good for most sites (included with stealth mode)
- **ISP** — dedicated IPs, good balance
- **datacenter** — cheapest, most likely to be blocked

## scraping patterns

### extract structured data

```typescript
await page.goto("https://example.com/products");
await page.waitForSelector(".product-card");

const products = await page.$$eval(".product-card", cards =>
  cards.map(card => ({
    name: card.querySelector("h2")?.textContent?.trim(),
    price: card.querySelector(".price")?.textContent?.trim(),
    url: card.querySelector("a")?.href,
  }))
);

return products;
```

### handle pagination

```typescript
const allItems = [];
let hasNext = true;

while (hasNext) {
  const items = await page.$$eval(".item", els =>
    els.map(e => e.textContent?.trim())
  );
  allItems.push(...items);

  const nextBtn = await page.$("a.next-page");
  if (nextBtn) {
    await nextBtn.click();
    await page.waitForLoadState("networkidle");
  } else {
    hasNext = false;
  }
}

return allItems;
```

### handle infinite scroll

```typescript
let previousHeight = 0;
while (true) {
  const currentHeight = await page.evaluate(() => document.body.scrollHeight);
  if (currentHeight === previousHeight) break;
  previousHeight = currentHeight;
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
  await page.waitForTimeout(2000);
}

return await page.$$eval(".item", els => els.map(e => e.textContent));
```

## scaling with concurrent browsers

launch multiple browsers for parallel scraping:

```typescript
const urls = ["https://site.com/1", "https://site.com/2", "https://site.com/3"];

const results = await Promise.all(
  urls.map(async (url) => {
    const browser = await kernel.browsers.create({ stealth: true });
    const pw = await chromium.connectOverCDP(browser.cdp_ws_url);
    const page = pw.contexts()[0].pages()[0];

    try {
      await page.goto(url);
      return await page.title();
    } finally {
      await pw.close();
      await kernel.browsers.deleteByID(browser.session_id);
    }
  })
);
```

## browser pools for high-throughput scraping

pre-warm a pool of browsers for instant acquisition. no cold-start latency.

```typescript
const pool = await kernel.pools.create({
  name: "scraper-pool",
  size: 10,
  stealth: true,
  proxy_id: "residential-proxy-id",
});

// acquire, scrape, release — browsers recycle instantly
for (const url of urls) {
  const browser = await kernel.pools.acquire(pool.id);
  // scrape...
  await kernel.pools.release(pool.id, browser.session_id);
}
```

## long-running scrapes

kernel supports sessions up to 72 hours. no charges for idle time — you can pause overnight and resume the next day from the same point.

```typescript
const browser = await kernel.browsers.create({
  stealth: true,
  timeout_seconds: 259200, // 72 hours
});
```

## tips
- always use stealth mode for scraping — it's on by default with recaptcha solving
- use profiles to stay logged in across scraping sessions
- set reasonable timeouts — don't leave browsers running forever
- respect rate limits — add delays between requests
- clean up browsers in finally blocks
- use browser pools for high-volume scraping to eliminate cold-start latency
