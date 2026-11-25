# Real Estate Scraper: What 1,000 Listings Cost, Whether You Can Get MLS Status and Days on Market, and Which Plan Fits

Scraping a listing page is the easy half. Getting a record you can run comps on (MLS status, days on market, sold date, price per square foot, lot size) is where real estate scraper projects stall, and where the monthly bill stops being predictable. Three things decide whether a setup works: the field schema, whether the portal actually exposes pending and sold inventory in your market, and what each listing costs once proxies and anti-bot surcharges are counted. Below is what ScraperAPI's credit system turns into per 1,000 listings, the field checklist a record has to pass, and the limits that live in parameter tables rather than on the sales page.

> **Quick pick.** Hobby at **$49/month** (100,000 credits, 20 concurrent threads) is the right starting point for a pilot on one metro or a handful of ZIP codes, provided you aren't re-fetching thousands of listings daily. Move to Startup at **$149/month** (1,000,000 credits, 50 threads) when daily refreshes span several ZIPs. Business at **$299/month** (3,000,000 credits, 100 threads) is for teams pulling a full metro every day, and it caps out faster than most people expect once anti-bot surcharges apply. 👉 [Check the current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The plans and what each credit allowance buys

| Plan | Credits per month | Concurrent threads | Price (USD/month) | Link |
| --- | --- | --- | --- | --- |
| Free | 1,000 on signup | 5 | $0 | [Start on the free credits](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20 | $49 | [See Hobby pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149 | [See Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | $299 | [See Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling, Professional, Advanced, Enterprise | 5,000,000 and up, or custom | Custom | Quoted | [See the higher tiers](https://www.scraperapi.com/pricing?fp_ref=coupons) |

List prices as published in October 2026. ScraperAPI revises them periodically, so confirm on the live page before you budget.

A credit is not one request. ScraperAPI prices by target: a standard page costs 1 credit, Amazon 5, Google and Bing 25, LinkedIn 30. When bypass logic fires on a site behind Cloudflare, DataDome or PerimeterX, 10 credits are added to the base cost of that request. The dashboard's Domain Cost Estimator is the authoritative figure for any single URL, and a `max_cost` parameter lets you reject a request before it spends more than you allow. Every response also carries a credit-cost header, which makes per-listing accounting possible without guessing.

Three billing rules matter for planning:

- Credits reset at each monthly renewal and unused ones do not carry forward.
- Hobby, Startup and Business return a 429 when credits run out. There is no overage charge and no queue; the request is simply refused. Pay-as-you-go, which continues past the allowance at a fixed per-credit rate under a cap you set, starts at the Scaling tier.
- Annual billing takes exactly 10% off, so Hobby works out to roughly $44.10 a month effective.

## What a real estate listing record has to contain

A record with price, beds and baths fills a search results page. It cannot produce a comp. The baseline worth testing any scraper against is the schema Redfin-focused tools return, because it maps to how valuations actually get argued:

- **`mlsStatus`** with the real values: Active, Pending, Coming Soon, Contingent, Sold. A pending sale is the closest thing to a comp that hasn't closed yet, and Coming Soon is your inventory pipeline.
- **`soldDate` and sold price**, separate from list price. Comps are closed sales with dates attached.
- **`daysOnMarket`**, which is how you tell a fresh listing from one that has sat through two price cuts.
- **`pricePerSqFt`**, **`sqFt`**, **`lotSizeSqFt`**, **`yearBuilt`**, **`hoaPerMonth`**, **`propertyTypeName`**: the adjustment variables in any CMA.
- **`listingAgent`** and **`listingBroker`** if lead generation is part of the plan, plus **`latitude`/`longitude`** and the listing **`url`** so records can be joined and de-duplicated later.

One caveat that rarely appears in marketing copy: whether pending and sold listings come back at all depends on the source's coverage in that market. If your product promises national sold comps, verify a few ZIPs in different states before you promise anything.

ScraperAPI sits one level below the turnkey actors. You send a URL, it handles proxies, retries and, where needed, rendering, and returns the page. It has structured endpoints for a limited set of targets, and Redfin is one of them, which matters if Redfin is your primary source. For Zillow or Realtor.com you either parse the HTML yourself or lean on the autoparse mode and check the output before trusting it downstream.

## What 1,000 listings actually cost

The arithmetic is simple once you accept that the credit multiplier, not the plan price, drives the bill. At the October 2026 rates the per-credit cost works out to $0.00049 on Hobby, $0.000149 on Startup and about $0.0001 on Business.

| Workload (1,000 listings) | Credits | Hobby | Startup | Business |
| --- | --- | --- | --- | --- |
| Plain pages, one fetch per listing | 1,000 | $0.49 | $0.15 | $0.10 |
| Protected site, bypass fires (1 + 10) | 11,000 | $5.39 | $1.64 | $1.10 |
| Search page plus detail page, bypass fires | 22,000 | $10.78 | $3.28 | $2.19 |

The gap between the first and second row is the whole budgeting problem. Whether a portal triggers the 10-credit surcharge changes your cost per thousand by 11x, and it isn't visible from the plan page. Run your actual target URLs through the Domain Cost Estimator before choosing a tier. 👉 [Check per-credit pricing against your own targets](https://www.scraperapi.com/pricing?fp_ref=coupons)

Scale it to a real job: a metro with 5,000 active listings, refreshed daily with a search page and a detail page each, is 10,000 fetches a day, or 300,000 a month. At 1 credit per fetch that fits inside Startup with room to spare. At 11 credits it becomes 3.3 million credits, which is past Business's 3,000,000 allowance and into the pay-as-you-go tiers. Incremental refreshes, where you re-fetch only listings whose price or status changed, cut that number by an order of magnitude and are the single biggest cost lever available.

For comparison, the per-record alternatives charge differently. The Apify real estate aggregator bills per event: $0.002 per listing on its Free plan and $0.0015 on Diamond, plus a small per-run start fee, so 1,000 finished records cost about $2.00, or $4.00 if you turn on detail pages, which are charged again at the same rate. Those figures come from a page dated April 2025 and may have moved. ScrapeHero's Realtor scraper bills 1 data credit per record, and its documentation doesn't publish a per-credit price, so you have to price it on their site.

The comparison isn't apples to apples. At Business rates, $1.10 buys you 1,000 protected page fetches that you still have to parse. Two dollars buys finished, normalized records from the aggregator. If your parsing pipeline already exists, credits are cheaper by a wide margin. If it doesn't, the per-record price is often less than the engineering time to build one.

## Why anyone pays for a scraping API instead of hitting the portals directly

Because the official doors are closed or narrow. Realtor.com has no public listing API, which is why every Realtor tool on the market is effectively an unofficial substitute. Redfin runs a Data Center API, but it has a waitlist and is aimed at enterprise users, which leaves independent developers and research teams looking for a third path.

The proxy question also has no single answer, and this trips people up. A Redfin-focused actor advertises HTTP-only operation with no proxies required. A Realtor.com actor ships with US residential proxies switched on by default because Realtor.com requires them. Anyone who tells you real estate portals uniformly need residential proxies, or uniformly don't, is generalizing from one site. Buying the proxy layer as a service is mostly about not maintaining per-target rules yourself.

## Six failures that only show up in production

1. **Sort parameters get rejected.** On the Redfin side, `newest` and `days-on-market-asc` are refused by Redfin's backend and now fail with an explicit input error. What's left is recommended order, price ascending or descending, square footage descending and lot size descending. If your product promise is "new listings today," you filter on list date yourself.
2. **The default status filter quietly narrows your results.** Leaving listing status empty returns active plus coming soon. Pending and sold have to be requested explicitly, and their availability varies by market.
3. **"Unlimited" isn't unlimited.** Setting `maxListings` to 0 paginates to a 50-page safety cap, not to the end of the inventory.
4. **Partial failure looks like success.** A multi-site aggregator only fails the run when every provider fails. One blocked site leaves the other sites' rows sitting in the dataset and no error in your pipeline. Monitor per-site counts, not just total rows.
5. **Deduplication is string matching.** Cross-site dedupe matches on offer type, country, ZIP and street, so the same house written two ways stays as two rows.
6. **AI extraction is capped by tokens per minute, not by ambition.** On Groq's free tier, 70B and 120B models run at roughly 6,000 TPM, which throttles full-page extraction to one or two listings per request. A Gemini flash model with a 4,000,000 TPM ceiling pulls 20 to 25 complete listings per page. Crawl cost also scales exponentially with sites multiplied by pages per site, so test one site and one page first.

On the ScraperAPI side, the equivalent failure is a 429. Exceeding your concurrency gets the request turned away with no charge, and exhausting credits does the same on Hobby through Business. Neither is billed, but neither is queued either, so a scheduled job that runs hot will silently produce gaps. 👉 [Compare the credit allowances before you size a plan](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The legal line

Scraping publicly visible listing data has a reasonable footing in US law. The Redfin actor's documentation cites hiQ Labs v. LinkedIn, decided in the Ninth Circuit on 18 April 2022, and the practical guidance across every tool reviewed here is the same: stay on logged-out, public pages, don't touch agent-only areas, and keep request rates polite. Agent names and phone numbers are a different category. Once you collect them for outreach, GDPR and CCPA style rules apply, and several tools say so explicitly in their own docs. Portal terms of service still bind you regardless of what a scraper can reach.

## How the turnkey alternatives compare

| Option | What you get | Coverage | Pricing model | Watch out for |
| --- | --- | --- | --- | --- |
| Apify Redfin scraper | Parsed Redfin records, 30+ fields, no HTML parsing | Redfin only | Apify event billing | `newest` sort rejected; pending/sold coverage varies by market |
| Apify real estate aggregator | Normalized records, cross-site dedupe | Realtor.com, Zillow, Redfin, Apartments.com, Zumper | ~$0.002/listing, $0.0015 on Diamond (page dated April 2025) | One location per run; type filters apply on only two of five sites |
| ScrapeHero Realtor scraper | Hosted app, Realtor.com records | Realtor.com | 1 data credit per record | Price and area returned as formatted strings; year built can be null |
| Open-source AI extractor | Portal-agnostic pipeline you host | Global, any language | Your own LLM token spend | TPM limits throttle output; cost scales with sites × pages |

The data-cleanliness difference is worth more attention than it usually gets. The Apify actors return price and square footage as numbers. The ScrapeHero example record returns `"$2,400,000"` and `"3,776 sqft | 7,405 sqft lot"` as strings, with a null year built. If your valuation model consumes these fields directly, one of those formats needs a cleaning layer first.

## Which one to pick

- **Redfin only, and you want finished records with no engineering:** the dedicated Redfin actor is the shortest path, and its field list is the best public documentation of what a listing record should hold.
- **Zillow plus Redfin plus Realtor.com in one run, small team, no parser:** the five-site aggregator at roughly $2 per 1,000 records is hard to beat on time-to-first-data, provided you keep runs to one location at a time.
- **You already parse, or you want control over the fetch layer:** ScraperAPI, because credits are metered per domain, the async API accepts batches of up to 50,000 URLs with retries over 24 hours, and the Redfin structured endpoint covers the source that matters most for comps.

Before you pay, confirm two things. First, run your real target URLs through the Domain Cost Estimator, since the 10-credit bypass surcharge is the difference between $0.10 and $1.10 per thousand. Second, check that whatever plan you choose survives a month of your actual refresh cadence, because unused credits expire at renewal and there's no overage forgiveness below the Scaling tier. If you want to validate both with real data first, the free 1,000 credits and 5 concurrent threads are enough for a few ZIP codes. 👉 [Pick a plan on ScraperAPI's pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)
