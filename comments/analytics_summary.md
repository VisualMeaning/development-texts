# Implementing Basic Web Analytics for the SMP

We would like to start tracking basic user activity on the Shared Meaning Platform -- how many users are visiting, which pages they visit, where they come from and so on. There are many, many web analytics platforms that we can integrate that offer these statistics. The purpose of this document is to briefly summarise our tracking goals, and then outline how well each platform fulfils those goals, before making a recommendation towards which platform we should select.

## Goals

The service we select should fulfil this (non-exhaustive) list of basic tracking requirements, as stated by the [ticket](https://app.productive.io/15481-visual-meaning/projects/130026/tasks/task/1726443?board=161977&filter=LTE%3D):

- how long a user's visit was
- what content they visited (which URLs)
- what device they were using
- which geography their IP reflected as being located in
- which browser they were using

### Time-on-page

The standard method of measuring time-on-page is to measure the time between a user arriving at one page on your site, and then navigating to another page on your site. However, if a user recieves a direct link to a single page (for example, a map), spends all their time on that one page, and never navigates to another page on the site, then no time-on-page stat is recorded as the analytics platform has no way of knowing when they left the page.

If the typical user flow for leaving a map is to close the browser tab, or do anything that's not visiting somehwere else on the VM site, then this would unfortunately make it quite difficult for us to know how long a user spends looking at a given map even if we're using an analytics service that records this stat.

### User identities

There is an additional requirement in the ticket:

- who visited (in instances where users logged in)

However the identity of a logged-in user is internal VM-only information that tracking services do not and should not care about. They track unique users, and *how* they do this is of interest because of privacy concerns (see **GDPR** below), but they do not know the VM identity of a user. And to be honest I'm not sure that *we* want to know this either thanks to the same privacy concerns.

### Additional considerations

**GDPR**: Is the service so exhaustive with its user tracking that it requires you to display a GDPR consent form to the user? Usually this involves collecting (and retaining) enough data about a unique user that it is possible to consistently identify them over a sustained period of time (> 1 day), which means the data collected falls into the realm of Personally Identifiable Information (PII), and thus GDPR. This is both undesirable from a UX point of view, and also potentially opens up a *massive* legal can of worms with respect to who is responsible for the collected data. Ideally we want to stay far away from services that fall under the scope of GDPR.

**Cost**: How much does the tracking service cost? Some services are free, but that's usually because they're collecting data for marketing purposes which has drawbacks elsewhere. Most solutions are paid unless you're hosting it yourself.

**Installation**: How easy is the tracking service to set up and install? The canonical easy example is Google Analytics, which requires you to include a single javascript tag in your base site page and then "just works". Anything with much more overhead than this is probably not worth the effort.

The size of the above javascript file is also a concern as it's one of the first things loaded when you visit a site, meaning that it can slow down initial loading times. This won't be included as an explicit evaluation criteria, however if a service has a particularly chunky tracking script it will be noted.

## Services

### Google Analytics

What passes for the industry standard because the barrier to entry is so low. Free, easy to install, and collects a ton of data about each user -- but this is designed to build a persistent identity that can be tracked across multiple sites and used by Google to target advertising, which means it falls under GDPR. Also the sheer pervasiveness of it is something of a drawback, since there are a lot of people who don't like Google being able to see what they do on the internet and who explicitly add Google Analytics tracking scripts to their adblockers.

| Criteria | Result | Notes |
| --- | --- | --- |
| **Tracking requirements** | All | Collects all of the required data, and then some. |
| **GDPR** | Required | Google collects information primarily for their own advertising purposes, and you have no control over what gets collected using their service. |
| **Cost** | Free |  |
| **Installation** | Good | Tracking script is bloated at 73k. |

**Summary**: Would be a reasonable solution if not for the amount of data collected from users, which is both massive overkill for what we want to do here and tremendously invasive. Avoid.

### Fathom 

[Link to demo](https://app.usefathom.com/share/krtexeuo/serverless+laravel)

Positions itself as an alternative to Google Analytics that offers much of the same functionality with none of the privacy concerns. Converts IPs to hashes (that  can't be backtraced to users) and then scrubs the hashes after 24 hours. Billed on page views, but we are exceedingly unlikely to exceed the cheapest tier of $14 per month for 100k page views. Has some handy additional features like uptime monitoring. Can bypass adblockers by configuring custom subdomain.

| Criteria | Result | Notes |
| --- | --- | --- |
| **Tracking requirements** | All | Device is tracked at a high level of desktop/phone/tablet but has no specific device info beyond that. |
| **GDPR** | Not required | Fathom's USP is that it doesn't require GDPR consent. |
| **Cost** | $14/mo for 100k page views | 7 day free trial available for testing purposes. |
| **Installation** | Good | Installed via script tag. |

**Summary**: A decent hosted solution that ticks all of the boxes as long as we don't want specific device information.

### GoatCounter

[Link to demo](https://stats.arp242.net/?hl-period=week&period-start=2021-06-30&period-end=2021-07-07&filter=&as-text=off&daily=off)

Driven by the same privacy requirements as Fathom, but with the additional requirement that the codebase be properly open source. One minor drawback compared to Fathom is that a GoatCounter unique user is always scoped to a path -- there is no way to aggregate up and get unique users site-wide. It's also not possible to bypass adblockers without self-hosting GoatCounter.

| Criteria | Result | Notes |
| --- | --- | --- |
| **Tracking requirements** | 4/5 | GoatCounter does *not* appear to track time-on-page. Also has same high-level device classification as Fathom. |
| **GDPR** | Not required |  |
| **Cost** | €5/mo for 100k page views | Free version available for non-commercial use. |
| **Installation** | Good | Installed via script tag. |

**Summary**: Cheaper than Fathom and has good policies around open source and access to data, as well as being the easiest service to self-host (if we had to). However, has some flabbiness around which stats are measured and how.

### Matomo

[Link to demo](https://demo.matomo.cloud/index.php?module=CoreHome&action=index&idSite=1&period=day&date=yesterday).

A more heavyweight competitor to Google Analytics. It *can* be configured to avoid GDPR consent forms, although given number of options around privacy it looks like it'd be quite easy to shoot ourselves in the foot here. Blurs the boundaries between web-tracking statistics and business intelligence tool, which in practical terms means it has a bunch of functionality we do not need and which make the user experience overly complicated.

| Criteria | Result | Notes |
| --- | --- | --- |
| **Tracking requirements** | All | Has so much data that we'd have trouble finding the things we care about. |
| **GDPR** | Not required | ...if configured correctly. |
| **Cost** | £29/mo for 50k page views | 21-day free trial. |
| **Installation** | Good | Installed via script tag, but extremely chunky at 192k. |

**Summary**: If we were a much larger organisation that was running some high-traffic website like an e-commerce platform, I think Matomo would be appropriate. As it is I think it is both expensive, and massive overkill.

### SimpleAnalytics

[Link to demo](https://simpleanalytics.com/simpleanalytics.com).

Has a very similar featureset to Fathom, although it appears to be a bit less mature with several features being marked as in Beta.

| Criteria | Result | Notes |
| --- | --- | --- |
| **Tracking requirements** | All | |
| **GDPR** | Not required | |
| **Cost** | £19/mo for 100k page views | 14-day free trial. |
| **Installation** | Good | Installed via script tag. |

**Summary**: It being identical to Fathom in all respects except for charging an additional £5 per month means there's no reason to use this over Fathom.

### Others

* StatCounter ([Link to demo](https://statcounter.com/demo/summary/)) - no device or time-on-page stats and appears to persistently track users, meaning it'd fall foul of GDPR.
* Count.ly - setup is complicated, pricing is opaque, and there's nothing on the website about privacy. The website doesn't even say what it tracks, although I suspect this is because you have to configure it all manually.

## Conclusion

Fathom and GoatCounter are the two most viable candidates. Fathom is more expensive but fulfils all of our requirements. GoatCounter is cheaper but does not track time-on-page -- however this may be a less-useful stat than it seems thanks to the likely path users will take through our site.
