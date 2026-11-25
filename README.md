# Airbnb data scraping: what breaks a DIY scraper, what one city costs in credits, and which ScraperAPI plan to buy (plan table + field checklist)

Airbnb's official API is closed to new applicants and caps at 300 results per search with 1,000 in total, so any serious coverage of a market means pulling the public site. DataDome decides whether that works. A plain Python request returns HTTP 200 wrapped around a challenge page instead of listings, and the occasional lucky request is worse than a clean block because it makes the build look finished. ScraperAPI bills an Airbnb page at **11 credits** rather than the standard 1, since bypassing DataDome adds a 10-credit surcharge.

Below: where a self-built scraper stalls, how many credits a city really costs, which fields force a second request per listing, and how to pick a plan from your refresh frequency instead of the plan names.

> **Short version.** Run the trial first and measure your own burn. 5,000 credits is roughly 450 Airbnb page requests, about one city at search-result level. Full detail on 2,000 properties runs near 25,000 credits per snapshot, so buy by allowance rather than by tier name. 👉 [Start the 7-day trial and read the current plan allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## ScraperAPI plans at a glance

Every tier is sold by monthly credit allowance, and the dollar figure on the pricing page moves with promotions. Treat the allowance as the stable thing and confirm the number before you pay.

| Plan | Verified capacity | Keep-scraping options | Best fit | Start here |
| --- | --- | --- | --- | --- |
| Free | 1,000 credits, 5 concurrent connections | None | Confirming that a proxy API returns real Airbnb listings before you spend anything | [Open the free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Entry paid tier | Auto-upgrade | One city, refreshed occasionally, search-level data only | [Check Hobby credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Higher allowance than Hobby | Auto-upgrade | Two or three cities on a monthly refresh | [Check Startup credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | Higher allowance again | Auto-upgrade | A regional market tracked weekly with detail pages | [Check Business credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Up to **200 concurrent threads** | Auto-upgrade and pay-as-you-go | Multi-city daily refreshes where a spike shouldn't stall the pipeline | [Check Scaling credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Larger allowance, same operating options as Scaling | Auto-upgrade and pay-as-you-go | Continuous price monitoring with historical backfill | [Check Professional credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | High-volume allowance | Pay-as-you-go | National coverage or an agency running several client markets | [Check Advanced credits and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 10,500,000 credits, **300 threads**, country-level geotargeting, unlimited analytics; custom trials reach 50M credits and 300+ threads with a Slack channel | Pay-as-you-go | Whole-continent datasets and agent-fed pipelines | [Talk to sales about a custom trial](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two operational details that don't show up until you're inside a billing cycle: your credit balance **resets on renewal** rather than rolling over, and pay-as-you-go is only available from Scaling upward. If you expect uneven months, that combination decides your tier more than the headline allowance does.

## Where a self-built Airbnb scraper actually breaks

The first request is easy. The two-hundredth is the problem.

DataDome sits in front of Airbnb and it does not return a clean 403 you can code around. It returns a page that parses as valid HTML with no listings inside. Once you add pagination, a second city, or any real volume, you collect CAPTCHAs, rate limits, and IP bans, and you're now maintaining residential proxy rotation, a browser fingerprint that survives scrutiny, CAPTCHA handling, and a parser that keeps working through Airbnb's markup changes. Each of those is a service you either buy or babysit.

The pagination ceiling compounds it. Airbnb's search results stop at roughly **270 listings per query**, which is a property of the website, separate from the 300-result cap on the official partner API. A city with a few thousand listings will not surrender them through one search URL. You split the price range into bands, re-run the search per band, and keep subdividing any band that still hits the ceiling. That's the technique every working Airbnb scraper uses, and it's also what turns a modest city into hundreds of page requests instead of a handful.

## The credit math per city

ScraperAPI charges 1 credit for a standard page. Sites behind Cloudflare, DataDome, or PerimeterX add 10 credits per request when the bypass is used, which puts Airbnb at 11. Parameters change the final number, so the Domain Cost Estimator in the dashboard is worth a look with your own URL before you commit to a plan size.

Working from 11 credits per page:

- **Search-level sweep, one city.** A 2,000-listing market needs at least eight price bands after splitting, and a few hundred page requests once you've walked each band. Call it 300 requests, about **3,300 credits**.
- **Full detail, same city.** One extra request per listing for amenities, house rules, host data, and reviews: 2,000 × 11 = 22,000 credits, plus the search sweep. Budget **25,000 credits** per snapshot.
- **Weekly refresh of four cities with detail pages.** 25,000 × 16 ≈ **400,000 credits a month**.
- **Daily refresh of a single city with detail pages.** 25,000 × 30 ≈ **750,000 credits a month**.

Per 100,000 credits you can afford roughly four full city snapshots or about thirty search-level sweeps. Those two figures differ by an order of magnitude, which is why the tier question is really a fields question. 👉 [Price out your own city volume against the plan allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Which fields cost you a second request

Search result pages carry enough to build a pricing model. Listing detail pages carry everything else. Deciding this before you buy is the single biggest cost lever.

| Available from search results | Requires the listing detail page |
| --- | --- |
| Listing ID and URL | Full amenity list, categorised (kitchen, bathroom, safety, entertainment) |
| Title and short description | House rules: check-in and check-out times, guest caps, pet and smoking policy |
| Nightly price and displayed total | Host profile, Superhost status, co-hosts, host highlights |
| Overall rating and review count | Review text with dates and ratings |
| GPS coordinates | Full image set, typically 15 to 40 photos with captions |
| Property and room type | Breadcrumb path (country → region → city) |
| Guest capacity | Tax-inclusive versus pre-tax totals, where Airbnb exposes both |

If you're tracking price movements across a market, the left column plus a scheduled rerun gets you a time series at a fifth of the cost. If you're scoring properties on amenities or running sentiment analysis on review text, you're buying detail pages and should size the plan accordingly.

## Airbnb-specific traps worth designing around

**Currency and locale default to one setting, and prices don't.** Set the currency and locale explicitly on every request. Mix them across a crawl and your price column silently holds dollars, euros, and yen in the same field, formatted differently.

**Displayed totals are not comparable totals.** Airbnb's headline price may exclude taxes and fees depending on the market. Pull the pre-tax and tax-inclusive breakdown where it exists; otherwise cross-border price comparisons are noise.

**The top price band needs a ceiling.** Price-range splitting assumes an upper bound. Pick something sane like 10,000 per night and check whether your market has listings above it, or luxury inventory quietly disappears from your dataset while the counts still look plausible.

**Nothing is stable.** Selectors break, layout shifts, and a parser that worked in March returns empty fields in June. This is the argument for buying the parsing layer along with the proxies.

## What ScraperAPI adds on the Airbnb side

Requests go out through a pool of **200M+ IPs** spanning residential, datacenter, and mobile, with full browser rendering and retry logic, which covers the fingerprint and rotation work described above. Geotargeting is country-level via `country_code`, so a Rome search can be issued from Italy rather than from a datacenter in Virginia.

For volume, the Async API takes large batches in parallel, retries failures automatically, and delivers results to a webhook, so a nightly city sweep doesn't need a script babysitting a loop. Useful parameters: `render` for dynamic pages, `premium` and `ultra_premium` to force residential and mobile proxies on the harder targets, `device_type` to request desktop or mobile layouts, `output_format` to get markdown or text instead of raw HTML, and `max_cost` to cap what any single request can consume. 👉 [See the Airbnb endpoint and its parameter set](https://www.scraperapi.com/solutions/airbnb-scraper?fp_ref=coupons)

## Compliance, briefly

Scraping publicly visible pages is broadly lawful, and the constraints that matter are about what's in those pages. Host names, profile links, and review author IDs are personal data under GDPR and CCPA, and listing descriptions and photographs are copyrightable, which brings the EU DSM Directive and US fair use into play. Keep personal data out of stored datasets where you don't need it, and don't republish listing copy or images verbatim. This is a design constraint, not legal advice.

## Who shouldn't buy a subscription

If you need a few dozen listings once for a school project or a single investment decision, a monthly credit plan is the wrong shape. The free tier's 1,000 credits covers about 90 Airbnb requests, enough to see whether the pipeline works and not much more.

Two reference points from the market, both stated on their own pages rather than tested here. Apify lists its Fast Airbnb Scraper at **$2.00 per 1,000 results** (page dated March 2025), which suits people who want a no-code actor and per-result pricing. Bright Data's guide states a ready-made marketplace dataset at **$0.0025 per record with a $250 minimum order** and 5,000 free records a month, on a page dated 2023, so treat those numbers as indicative of the model rather than current quotes. A one-off 20,000-listing historical pull can land cheaper as a dataset purchase than as a month of credits, and there's no shame in that.

Oxylabs is the other name worth knowing if batch size is your constraint: its Web Scraper API accepts up to 1,000 Airbnb URLs in a single request, and its 7-day trial is capped at 10 URLs before it starts returning rate-limit errors.

## Choosing by profile

**One city, search-level data, monthly refresh.** You need roughly 3,500 credits a month. Start on the smallest paid tier that clears it, and note that Hobby and above can auto-upgrade if a month runs hot.

**Four to six cities, weekly refresh, detail pages included.** Your ceiling is around 600,000 credits a month. Buy on allowance, and pick Scaling or above so pay-as-you-go absorbs a spike instead of pausing mid-crawl.

**Daily monitoring across many markets, or feeding an AI agent.** Look at Advanced or a custom Enterprise trial, where the thread count and geotargeting breadth matter as much as credits.

Before you pay, confirm four things: your real per-URL credit cost from the Domain Cost Estimator, whether you need detail pages at all, whether your usage pattern requires pay-as-you-go (Scaling and up), and the current dollar figure on the pricing page, since it changes with promotions while the credit allowance doesn't. 👉 [Pick the plan that covers your monthly city count](https://www.scraperapi.com/pricing?fp_ref=coupons)
