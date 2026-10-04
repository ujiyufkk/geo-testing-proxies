# proxies for geo testing: how to check any site from 90+ countries with city, ZIP and ISP-level targeting, starting at $15

Type that phrase into Google and you get two unrelated species of result. One is a browser extension built for ad-ops people — pick a location from a dropdown, see the creative the way a person in Berlin sees it. The other is a residential proxy network where you build the geo test yourself, scripted, from wherever your test runner happens to live.

Both are legitimate. They cost differently, and they fail differently. The useful question isn't "which is best" but "am I running twenty manual checks a week, or five hundred automated ones?"

Most people searching this term are doing something specific: confirming that a price, a currency, a shipping option, a banner or a search result actually changes when the visitor changes country. That's the job. Everything below is about doing it without guessing whether your exit IP is really in the city it claims.

## What geo testing actually demands from a proxy

Before comparing providers, it helps to be honest about the requirements, because they rule out a lot of cheap options fast.

- **Residential origin, not datacenter.** A datacenter IP from Frankfurt often gets the wrong page, the wrong price, or a CAPTCHA. You then spend an hour debugging your own test instead of the site.
- **Depth below country level.** Country targeting answers "does this differ by market". City, state and ZIP targeting answers "why does Denver see a different promo than Seattle", which is usually the real question.
- **Session control.** Price comparison wants a fresh IP per request. A checkout or login flow wants the same IP held for the whole sequence. A proxy that can only do one of those will limit you.
- **Automation without a desktop client.** If the test runs in CI, or in a cloud container, an app that needs a GUI installed on your laptop is a dead end.
- **Clean IPs.** If the pool is shared with every scraper on earth, the target site has already flagged half of your exits before you sent the first request.

## Why the VPN on your laptop quietly fails at this

A consumer VPN gives you a handful of cities, usually one per country, and that list rotates without telling you. There's no ZIP-level anything, and no way to ask for a specific ISP. Worse, VPN exit ranges are small and heavily documented, so sites that do any real geo-IP work already treat them as suspicious. You end up testing the VPN's blocklist rather than the site's targeting logic.

There's also the session problem. VPNs switch exits mid-session when a node gets overloaded. For a one-page currency check that's survivable. For a five-step checkout across three countries, you'll get results you can't reproduce, which is the same as having no results.

## 9Proxy's targeting model, and why it matters for this job

9Proxy is a residential proxy network with 20M+ IPs across 90+ countries, split into two different products that behave nothing alike. That split is the first thing to understand, because it determines whether your geo test is easy or annoying.

**Residential Proxy by IPs** gives you a fixed batch of residential IPs with unlimited bandwidth. Each IP lives a few hours up to roughly 24 hours, unused IPs don't expire, and there's no natural rotation — you rotate through an auto-rotation feature on selected ports if you want it. Authentication and forwarding run through 9Proxy's desktop app.

**Residential Proxy by GB** charges by traffic and lets you generate unlimited endpoints. Sessions can be sticky or rotating, authentication is username/password or IP whitelisting, and it works straight from the browser dashboard with no app involved.

For geo testing, the second one is almost always the right answer, and the reason is the targeting string. GB plans put the location inside the proxy username:


<subuser>-country-<cc>-st-<state>-city-<city>-isp-<isp>-sst-<minutes>-ssid-<id>


| Parameter | Example | What it does |
| --- | --- | --- |
| `country` | `country-de` | Routes through Germany |
| `st` | `st-ohio` | Narrows to a state or region |
| `city` | `city-newyork` | City-level targeting (use underscores for spaces) |
| `isp` | `isp-as22773_Cox_Communications_Inc.` | Filters by ISP or ASN |
| `sst` | `sst-15` | Sticky session length in minutes |
| `ssid` | `ssid-test3` | Distinct session ID so parallel tests get different IPs |

That last pair is the part people underuse. `sst` holds an IP long enough to walk a multi-step flow; `ssid` lets you fire off the same configuration ten times and get ten different IPs instead of ten requests through one. Ten city checks in one script, no serialisation, no collisions.

One caveat worth knowing up front: stacking filters shrinks the pool. Country plus city plus ISP on a small market can leave you waiting on availability. Start at country level, add `st` or `city`, and only reach for `isp` when the thing you're testing is genuinely ISP-dependent — carrier bundles, peering-specific content, that sort of edge case.

## Sticky or rotating: pick by what you're testing

| Test type | Mode | Config |
| --- | --- | --- |
| Currency, pricing, stock checks | Rotating | Country + city, no `sst` |
| Regional ad creative / landing page | Rotating | Country, one request per check |
| Cart, login, promo redemption | Sticky | `sst-15` to `sst-30` |
| Parallel checks across many cities | Sticky + `ssid` | One `ssid` per city |
| SERP differences between two nearby metros | Rotating | Country + `city` (and `st` if needed) |
| Carrier or ISP-specific pages | Rotating | `isp-<asn>` |

If your test needs the site to think nothing changed between step one and step five, sticky. If you're sampling, rotating. Mixing them inside one script is fine and normal.

## Which plan you actually want for geo testing

The instinct is to buy IPs, because "unlimited bandwidth" sounds like the safe choice. For geo work it usually isn't.

A geo check is a few hundred kilobytes: load the page, read the price, maybe screenshot it. You don't need unlimited bandwidth. What you need is a lot of different exits in a lot of different cities, and IP-based plans give you a fixed batch that you'd have to burn through and replace as their hours run out. GB plans bill exactly that pattern — small requests, heavy rotation — and the endpoints themselves are unlimited.

Geekflare's review makes the same split: the GB model suits high-rotation, low-bandwidth work such as ad verification, geo-checking and API polling, while IP plans suit long-running sessions where bandwidth is hard to predict. Geo testing lands firmly in the first bucket.

Bundles — IPs plus a traffic allowance — make sense if you're doing geo checks and also running logged-in account work from the same dashboard. If it's purely geo, a mid-size GB package is cheaper and simpler.

👉 [Compare 9Proxy's GB plans and start with a small package](https://bit.ly/9-Proxy)

## 9Proxy plans and pricing in full

Three product lines, all publicly listed. One important note first: on 1 June 2026, 9Proxy adjusted pricing for IP-based and bundle packages — its first increase since launch — while GB-based pricing stayed exactly as it was. IP-based and bundle figures below reflect the post-adjustment structure; the checkout page is always the authoritative number.

### Residential Proxy by GB

| Package | Price per GB | Total | Validity |
| --- | --- | --- | --- |
| 5 GB | $3.00 | $15 | 180 days |
| 50 GB (+5 GB bonus) | $2.10 | $105 | 180 days |
| 100 GB | $1.50 | $150 | 180 days |
| 200 GB | $1.00 | $200 | 180 days |
| 1,000 GB | $0.80 | $800 | 180 days |
| 2,000 GB | $0.75 | $1,500 | 180 days |
| 3,000 GB (Enterprise) | $0.72 | $2,160 | No expiry |
| 10,000 GB (Enterprise) | $0.68 | — | No expiry |

The 180-day clock on standard GB traffic is generous for testing work, and it's why project-based teams don't get punished for a quiet month. Enterprise packages drop the expiry entirely, and add team mode (one owner plus up to five members), per-member traffic controls, activity logs and shared bandwidth that doesn't expire inside the team.

👉 [Check the current GB-based packages and validity terms](https://bit.ly/9-Proxy)

### Residential Proxy by IPs

| Package | Price per IP | Total | Traffic |
| --- | --- | --- | --- |
| 100 IPs | $0.24 | $24 | Unlimited |
| 500 IPs | $0.144 | $72 | Unlimited |
| 1,000 IPs (+500 bonus) | $0.084 | $126 | Unlimited |
| 2,500 IPs | $0.084 | $210 | Unlimited |
| 5,000 IPs | $0.072 | $360 | Unlimited |
| 15,000 IPs | $0.048 | $720 | Unlimited |
| 25,000 IPs | $0.035 | $863 | Unlimited |
| 50,000 IPs | $0.029 | $1,438 | Unlimited |
| 100,000 IPs | $0.02 | $2,300 | Unlimited |
| 500,000 IPs | $0.015 | $8,625 | Unlimited |

Unused IPs don't expire, which softens the "hours to 24 hours per IP" lifetime considerably — you're not racing a clock, you're spending inventory.

### Bundle packages

| Bundle | Contents | Price |
| --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 |
| Popular | 1,500 IPs + 50 GB | $180 |
| Pro | 5,000 IPs + 500 GB | $720 |

Officially quoted entry pricing on the network sits at $0.015 per IP and $0.68 per GB at the top volume tiers. Payment options cover cards, Alipay, Apple Pay, Google Pay and crypto including USDT, BTC, ETH and LTC. 9Proxy also advertises a 5% discount for users who arrive through a referral — worth checking what your cart shows before you confirm.

👉 [See all 9Proxy packages and pick a plan](https://bit.ly/9-Proxy)

## Setting up a geo test, start to finish

1. **Create an account and buy a GB package.** For a first geo project, the 5 GB tier at $15 is enough for thousands of page loads. Testing 30 countries daily costs pennies at that volume.
2. **Create a sub-user.** In the dashboard, make a sub-account and assign traffic to it. This is the identity your requests authenticate with, and it keeps per-project usage countable.
3. **Choose your authentication.** Username/password is the flexible option. IP whitelisting is the one you want in CI, because there's no secret to inject — whitelist the runner's IP and any request from it routes automatically.
4. **Generate endpoints.** The Proxy Generator takes country, state, city, ZIP and ISP, plus sticky or rotating mode, and exports as `.txt` or `.csv` with ready-made code samples. Full country list, bulk export, straight import.
5. **Verify before you trust.** Always confirm the exit is where you asked:


curl -x yourproxyhost:yourport \
  -U "subuser-country-de-city-berlin-sst-15:subuser_password" \
  https://ipinfo.io/json


Read the returned country, city and ISP. If they don't match your target, the problem is the proxy config, not the site. If they match but the page still shows the wrong currency, it's usually session reuse, a cached CDN edge, or the site serving by language header rather than IP — different bug, different fix.

## Automating geo checks across a lot of locations

The pattern that scales is boring, which is a compliment. One sub-user per environment (staging checks, production checks), an IP whitelist for whatever machine runs the suite, and one proxy string per target location built from country/city parameters. With `ssid` you can run twenty cities concurrently without them collapsing onto the same exit IP.

Because the GB product works entirely from the dashboard — no desktop app, no local port forwarding — it drops into a container or a cloud runner as a plain HTTP proxy string. If you're driving headless Chrome, use the HTTP/HTTPS endpoint rather than SOCKS5; authenticated SOCKS5 is the one configuration Chromium handles badly.

There's a Public API for session control and usage stats if you'd rather not scrape your own dashboard, and SOCKS5 is supported natively for tools that want it — anti-detect browsers, proxychains, custom Python scripts.

👉 [Set up geo endpoints from the dashboard](https://bit.ly/9-Proxy)

## What 9Proxy won't do for you

Worth reading before you buy, because these are real constraints rather than marketing caveats.

**IP-based plans need the desktop app.** Local port forwarding, optional proxy authentication, a GUI to filter and forward proxies. That's fine on a workstation. It's painful for multi-device setups and awkward for headless CI — one review calls the mandatory app the network's main friction point. If automation is the whole point, take GB.

**No documented mobile-carrier targeting.** Location targeting runs country → state → city → ZIP → ISP. If your geo test needs a specific mobile operator's view — operator-targeted ad campaigns, carrier billing pages — a purpose-built geo-testing tool with carrier locations is the correct instrument.

**The pool is smaller than tier-one providers.** 20M+ residential IPs is a real network, but Bright Data-scale pools run to 150M+. For Tier 1 and Tier 2 targets, budget residential networks hold up well; for hard Tier 3 sites, expect more retries.

**Streaming platforms are hit and miss.** One review found e-commerce and sneaker-site work consistently successful while Netflix detected the connection. If your "geo testing" is really "watch another region's catalogue", rent a tool built for that.

**Trials aren't a self-serve button.** 9Proxy grants limited trials to new users subject to availability, and you generally have to ask support — and specify whether you want an IP-based or GB-based trial. Budget $15 instead and skip the queue.

**Speed is adequate, not headline.** Reported throughput sits around 50–100 Mbps, which is plenty for page rendering and API checks and not what you'd pick for bulk media transfers.

## When a dedicated geo-testing tool beats a proxy network

Different tools for different jobs, and the honest answer depends on whether a human or a script is doing the looking.

| Tool type | Examples | Strength | Weakness for geo work |
| --- | --- | --- | --- |
| Browser extension geo-testing | Pangeo / GeoEdge | 150–210 locations, 30–70+ mobile carriers, affiliate link and ad tag validators, ad QA workflow | Manual clicking, not scriptable at scale, licence priced for teams |
| Geo-IP test proxy network | WonderProxy | Purpose-built for localisation QA, tokens instead of passwords, official Sauce Labs integration for Playwright runs | Narrower use case, not a general scraping pool |
| General residential network | 9Proxy | Cheap per GB, city/ZIP/ISP targeting, sticky and rotating, unlimited endpoints, drops into any script | No toolbar, no carrier targeting, you build the workflow |

If your job is a media buyer eyeballing creatives in twenty markets, the extension wins on workflow. If your job is 500 automated checks a night with results in a CSV, the network wins on price and control — and at $1.00–$3.00 per GB you're not paying enterprise money to find out whether your test suite works.

## Questions that come up a lot

**How many countries can I actually test from?** 9Proxy lists 20M+ residential IPs across 90+ countries, with targeting to state, city, ZIP and ISP depending on market depth.

**Do I need the desktop app?** Only for IP-based plans. GB-based plans run from the dashboard with username/password or IP whitelisting.

**What happens to unused GB?** Standard packages carry 180-day validity. Enterprise packages have no expiry at all.

**Can I run sticky and rotating in the same project?** Yes — `sst` decides stickiness per session, so you can hard-code the mode into each test's proxy string.

**Is a free trial worth chasing?** Support grants limited trials subject to availability, but the 5 GB package at $15 is a faster path to a verified answer.

**Where's the ceiling on cost?** A modest daily geo suite — say 40 locations, 20 checks each, 400KB per check — burns well under a gigabyte a day. Small GB packages last a long time at that rate.

## The short version

For geo testing, buy traffic, not IPs. A GB-based plan gives you unlimited endpoints, real control over country, state, city, ZIP and ISP, sticky sessions long enough for multi-step flows, and parallel sticky IPs via `ssid` — all from a dashboard that doesn't require installing anything. Start at $15, verify your exits with an IP lookup before you trust a single result, and scale once the checks are reproducible.

👉 [Start a 9Proxy account and test your first location](https://bit.ly/9-Proxy)
