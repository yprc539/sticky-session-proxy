# sticky session proxy: how to hold one IP through a whole login or checkout, and which 9Proxy plan to run it on

A login that fails on the second request. A cart that empties itself between the product page and payment. A dashboard that throws you out after thirty seconds. In most of those cases the fingerprint is fine and the headers are fine. The exit IP moved, and the site noticed that the cookie it handed to one address came back from another.

That is the entire problem a sticky session solves. Not anonymity, not pool size. Just: keep the same exit IP for the length of one job.

Below is what sticky actually means in practice, how 9Proxy builds it in a username string, how long to hold a session, and the full current plan list so you can pick the right package instead of guessing.

## Sticky and rotating are two different products, not two quality levels

Rotating gives you a fresh exit per unit of work. Depending on the endpoint, that unit is one request or a short window, so consecutive requests can leave from different cities.

Sticky gives you one exit IP and holds it for a fixed window. Every request inside that window goes out from the same address.

The window itself is worth calibrating against what the market treats as normal, because it varies a lot:

- SOAX runs sticky sessions from 10 seconds to 3,600 seconds, with a default of 360 seconds [2]
- Decodo offers 1, 10, 30, or 60 minutes by preset, plus custom durations up to 24 hours [7]
- Massive defaults to a 15-minute TTL with a maximum of 1,440 minutes and doesn't extend it on activity [6]

So "sticky" can mean anything from ten minutes to a day, and the useful number depends on your job, not on the provider's default.

Where rotators win is a workload made of many small independent fetches, where each request is its own event and nothing needs to remember the last one. A browser visit is the opposite of that.

## Why one page load is one identity, not many requests

A single page load is a document, then dozens of subresources, then XHR and fetch calls, often a WebSocket. Three pieces of state tie those requests together and all three assume the exit stays put [5]:

**Cookies.** The session cookie the site set on the first response gets presented on every request after it. The site issued that cookie to whoever arrived on the first IP.

**Session tokens.** A CSRF token, an auth token, or a server-side session ID is bound to the context it was created in.

**The Referer chain.** In-site navigations carry the previous URL, so requests form an ordered path rather than a set of unrelated hits.

Rotate the exit in the middle of that visit and you hand the site one visitor whose IP contradicts its own cookies. The cookie says this browser arrived from address A a minute ago; the request carrying it now leaves from address B, in a different city. Nothing in the browser changed, but the story stopped making sense, and that contradiction is cheaper for a site to read than any in-page fingerprint [5].

### Workflows that actually need sticky

- Logging into accounts and staying logged in (ad accounts, marketplaces, social profiles)
- Checkout and cart flows, where a mid-session IP swap usually kills the order
- Multi-step forms, onboarding flows, anything with a wizard
- Authenticated API polling where the token is bound to the originating address
- Managing a set of browser profiles that each need to look like one consistent visitor

### Workflows that don't

- Bulk SERP collection where every query is independent
- Price checks against stateless endpoints
- High-volume scraping where no request needs to remember the last one

If your job falls in the second list, sticky is overhead, and a rotating endpoint is the cheaper and often faster answer.

## How 9Proxy splits the two models

9Proxy sells residential access two ways, and the split matters for sticky work because the two models behave differently at the session level [4].

**Residential Proxy by IPs.** You buy a fixed number of IPs, pay per IP, and bandwidth is unlimited while the IP is active. IPs last anywhere from a few hours to roughly 24 hours naturally, and unused IPs never expire. There's no natural rotation, though an Auto Rotation Proxy can rotate at custom intervals on selected ports. The catch: this model runs through the 9Proxy desktop app, which does local port forwarding, so it's a workstation setup rather than a drop-in endpoint. Authentication is handled by the app, optionally with proxy auth on top.

**Residential Proxy by GB.** You buy bandwidth and generate as many endpoints as your balance allows. IPs rotate automatically per request or per session, you choose rotating or sticky mode, and you authenticate either with username and password or an IP whitelist, straight from the dashboard with no app installed. Package validity is 180 days, unlimited on enterprise tiers.

That's the fork in the road. On the GB model, sticky is a parameter you set. On the IP model, persistence is the product itself: you're holding a specific residential IP until it dies on its own, which is often exactly what a long-lived account session wants.

## Building the sticky session string

On the GB model, everything lives in the username. The general format is [1]:


<sub-user>-country-<country_code>-st-<state_code>-city-<city_name>-isp-<isp_code>-sst-<session_time>-ssid-<session_id>


Not every segment is required. Here's what each one does:

| Segment | Example value | What it controls |
| --- | --- | --- |
| Sub-user's username | `subaccount` | Your sub-account name, issued in the dashboard |
| `country` | `country-us` | Target country by two-letter code |
| `st` | `st-ohio` | State or region filter |
| `city` | `city-newyork` | City-level targeting; use underscores for names with spaces |
| `isp` | `isp-as22773_Cox_Communications_Inc.` | ISP or ASN filtering |
| `sst` | `sst-15` | Session length in minutes; required for sticky |
| `ssid` | `ssid-ID1` | Unique session ID, needed for parallel sticky sessions |

Working examples from the docs [1]:


# US IP held for 15 minutes
subaccount-country-us-sst-15

# Hanoi IP, sticky for 20 minutes
subaccount-country-vn-city-hanoi-sst-20

# Parallel sticky sessions from the same config
subaccount-country-us-sst-15-ssid-id1
subaccount-country-us-sst-15-ssid-id2


The `ssid` is the part people miss. Each unique `ssid` gets you a different IP even when every other parameter is identical, which is how you run twenty sticky identities off one configuration instead of twenty separate setups [1]. To force an IP change early, don't wait for the timer, just change the `ssid`.

You can also skip the string entirely and pick **Sticky Session** in the dashboard's Proxy Generator under User-Pass Authentication, where you set the number of minutes in the panel and copy the resulting host, port, username, and password [3].

9Proxy's own summary of the combinations is compact: `country` alone for fastest random IPs, `country` plus city or ISP for geolocated rotation, `country` + `sst` for a single sticky device, and `country` + `sst` + `ssid` when you need stickiness across several devices or apps [1].

### How long should `sst` be?

The docs work in minutes and use 15 and 20 in their own examples [1]. Directory listings for 9Proxy reference sticky windows up to about 30 minutes, but the documentation doesn't publish a hard ceiling, so treat the upper bound as something to test with your own config rather than a spec sheet number.

Practical guidance that holds regardless: match the window to the flow. A login plus a two-minute session doesn't need an hour. A long dashboard session that dies at minute 20 is useless, and if your work genuinely needs a stable address for hours, the GB model's sticky cap may not be the right tool at all. That's when the IP-based model earns its place, since residential IPs there stay active from a few hours up to roughly 24 hours [4].

## Full plan list and current prices

One thing worth flagging before the tables: 9Proxy adjusted IP-based and bundle pricing on June 1, 2026, while GB-based prices were left alone. Several third-party pages still publish the old numbers, so if you see $20 for 100 IPs somewhere, that's the pre-adjustment rate [10][11]. The IP and bundle figures below are the post-adjustment ones.

All of these are balance-based one-off purchases, not monthly subscriptions. IP packages don't expire while unused; GB packages carry 180-day validity unless you're on an enterprise tier.

### Residential proxies by IP

| Package | IPs | Price per IP | Total | Notes |
| --- | --- | --- | --- | --- |
| Entry | 100 | $0.24 | $24 | Unlimited bandwidth per IP, unused IPs don't expire |
| Small | 500 | $0.144 | $72 | Solo operators running light multi-accounting |
| Most popular | 1,000 + 500 bonus | $0.084 | $126 | Effectively 1,500 IPs |
| Mid | 2,500 | $0.084 | $210 | Several verticals at once |
| Agency | 5,000 | $0.072 | $360 | Mid-scale price and SEO stacks |
| Large | 15,000 | $0.048 | $720 | Regional teams |
| High volume | 25,000 | $0.035 | $863 | Resellers, automation labs |
| Bulk | 50,000 | $0.029 | $1,438 | Platform-level operations |
| Business | 100,000 | $0.023 | $2,300 | Enterprise volume tier |
| Business | 200,000 | $0.021 | $4,140 | Enterprise volume tier |
| Business | 500,000 | $0.018 | $8,625 | Enterprise volume tier |

Every row above is covered by one purchase flow, so 👉 [check the current IP package pricing on 9Proxy](https://bit.ly/9-Proxy) before you commit to a volume tier.

### Residential proxies by GB

| Package | Price per GB | Total | Validity |
| --- | --- | --- | --- |
| 5 GB | $3.00 | $15 | 180 days |
| 50 GB + 5 GB bonus | $2.10 | $105 | 180 days |
| 100 GB | $1.50 | $150 | 180 days |
| 200 GB | $1.00 | $200 | 180 days |
| 1,000 GB | $0.80 | $800 | 180 days |
| 2,000 GB | $0.75 | $1,500 | 180 days |
| 3,000 GB | $0.72 | $2,160 | Unlimited |
| 6,000 GB | $0.70 | $4,200 | Unlimited |
| 10,000 GB | $0.68 | $6,800 | Unlimited |

The GB model is the one where sticky sessions are a username parameter, so if sticky is the whole reason you're here, 👉 [start with a GB-based 9Proxy package](https://bit.ly/9-Proxy) and set `sst` in the string.

### Bundle packages

| Bundle | Contents | Price | Notes |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | Mixed workloads, client pilots |
| Popular | 1,500 IPs + 50 GB | $180 | Teams running both stable accounts and burst traffic |
| Pro | 5,000 IPs + 500 GB | $720 | One all-in pack for larger operations |

Bundled traffic is valid for 180 days, which suits irregular project work better than a monthly window that expires whether you used it or not.

## Which package fits a sticky-heavy workload

The decision comes down to whether you're holding a few long identities or many short ones.

**One or two accounts, long sessions.** 100 IPs at $24 gets you a pool of individually stable addresses with unlimited bandwidth. You're not paying for sticky parameters, you're paying for addresses that stay alive for hours.

**Twenty to fifty parallel browser profiles.** The 500 IP package at $72, or the 1,000 + 500 package at $126 if you expect to grow. Per-IP cost drops from $0.24 to $0.084 across that jump, which is the steepest part of the curve.

**Agency work with a mix of sticky logins and bursty collection.** The Popular bundle at $180 is the sensible default: 1,500 IPs for the identities, 50 GB for the jobs that don't care about persistence.

**Mostly rotating, a handful of sticky.** Buy a small IP pack for the sticky accounts and put everything else on a GB plan like the 200 GB tier at $200. You'll pay $1.00 per GB there instead of $3.00 at the entry tier, and the 180-day validity means uneven project schedules don't burn balance.

Before any of it, 👉 [set up a 9Proxy account](https://bit.ly/9-Proxy) so you can test one sticky session string against your actual target site rather than a demo endpoint.

## What sticky does not fix

Worth being clear about, because sticky sessions get sold as a cure-all:

**A dirty IP is still dirty.** Sticky keeps you on one address; it says nothing about whether that address is already flagged. Success rates and pool cleanliness are a separate question from session persistence.

**Residential IPs drop on their own.** A Russian-language forum thread on searchengines.guru includes a user describing a test purchase of 100 IPs, using the Windows app to expose them as local ports for a multi-accounting tool, and noting that individual IPs lasted around three hours before going offline. Your retry logic still matters; sticky stops your schedule from rotating, not an ISP from disconnecting.

**Fingerprints are a different axis.** The IP that holds the cookies being the IP that received them makes your request history internally consistent. It doesn't make your browser look like a real Chrome install.

**Geo-targeting and sessions interact.** Change the targeting and you're effectively asking for a different identity, so use a distinct `ssid` per targeting configuration rather than reusing one across countries.

Latency is the other honest tradeoff. ProxyLook's directory entry lists 9Proxy at 3.9 out of 5 with a roughly 97% success rate, 20M+ residential IPs across 90+ countries, and an average response time around 1,300 ms. The company itself advertises 99.95% uptime, HTTP and SOCKS5 support, and targeting down to country, city, ZIP code, and ISP. If your workflow is latency-sensitive, that response time is a real consideration, not a footnote.

## Setup checklist

1. Buy a package and open the dashboard.
2. For GB-based sticky sessions, go to Residential Proxies by GB → Get Proxy and pick User-Pass Auth, or choose IP Whitelisting if you'd rather not manage credentials.
3. Select Sticky Session and set the duration in minutes.
4. If you need more than one sticky identity, don't create more sub-users. Append a different `ssid` per instance.
5. Test the exit IP with an IP checker on several consecutive requests and confirm the address holds for the whole window.
6. Force rotation early by changing the `ssid`, not by waiting out the timer.
7. For IP-based packages, install the desktop app instead, since that model routes through local port forwarding rather than a direct endpoint.

## Quick answers

**Is a sticky session the same as a static residential proxy?** No. Sticky pins an IP temporarily for a session window. Static or dedicated IPs are yours for longer. 9Proxy's IP-based packages sit closer to the second category, with addresses lasting hours to about 24 hours [4].

**Does sticky cost extra?** Not on the GB model. It's a parameter in the username string, not a paid add-on.

**Can I run many sticky sessions at the same time?** Yes, via `ssid`. Each unique ID gets a different IP from the same configuration [1].

**Do IP-based plans support sticky sessions?** Not in the username-string sense. The address is stable by nature there, and rotation instead happens through the Auto Rotation Proxy on selected ports at custom intervals [4].

**What happens if the IP dies mid-session?** You reconnect and get a new one. Build retries around that rather than assuming persistence, because residential addresses aren't contractually bound to stay up.

If you'd rather skip the theory and just run a test, 👉 [grab a 9Proxy package and try a sticky string against your target site](https://bit.ly/9-Proxy). One `sst` value, one `ssid`, five requests to an IP checker. That tells you more about whether sticky solves your problem than any comparison table will.
