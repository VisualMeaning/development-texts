# Domain hosting 

**2021-10-25**

Visual Meaning domains are currently registered with a domain registrar called 123 Reg (previously Domainmonster, who were bought by 123 Reg). Domain registrars typically offer cheap registration prices on signup and then gradually crank up the rate every time you renew, and 123 Reg are no exception. As several of our domains are up for renewal in mid-November, I have investigated potential alternatives to getting price-gouged by 123 Reg.

## VM domains

The following domains are hosted on 123 Reg.

| Domain | Expires | Used? |
| --- | --- | --- |
| transformation.delivery | 2021-11-11 | Yes |
| systemsthinking.games | 2021-11-11 | No |
| mapping.systems | 2021-11-15 | Yes |
| rich.pictures | 2021-11-15 | No |
| opmodel.guide | 2021-11-15 | Yes |
| meaning.guide | 2022-09-05 | Yes |
| systemic.world | 2022-09-09 | No |
| ecosystem.guide | 2023-05-09 | Yes |
| visual-meaning.com | 2024-05-19 | Yes |

As far as I can tell this is *all* of the VM domains, since it matches well with Martin's [map of VM web presence](https://www.plectica.com/maps/EJUL97BS7/edit/XMOMIPAAO) in Plectica.

`opmodel.guide` is hooked up to Cloudflare DNS, which handles routing to subdomains such as `policing.opmodel.guide`, `highways.opmodel.guide` and so on. It is also worth noting here that `eu.opmodel.guide`, `iam.opmodel.guide` and `ktd.opmodel.guide` all route to old maps served via FTP hosted by a provider called eUKHost rather than our standard Heroku setup. However, none of these services handle domain registration. Only 123 Reg does that. 

**DNS for all following domains is set up in 123 Reg directly**.

`ecosystem.guide` is our [production](https://eom.ecosystem.guide) and [staging](https://staging.ecosystem.guide) environment for the SMP.

[visual-meaning.com](https://visual-meaning.com) is our brochure website and primary domain for emails etc.

[meaning.guide](http://meaning.guide) is a essentially a blogging website on the theory and practice behind VM, linked to from the main site at `visual-meaning.com`. It doesn't have an SSL cert, which is exciting.

[transformation.delivery](https://transformation.delivery) is a zoomable version of the big map that's pinned to the wall above my (AL's) desk. This isn't hosted on Heroku and appears to be another FTP map similar to the eUKHost stuff on `opmodel.guide`.

`systemsthinking.games` fails to resolve.

`mapping.systems` links to the [purpose tool](http://eom-prototype.mapping.systems), hosted in Heroku. 

`rich.pictures` and `systemic.world` are what I'd describe in technical terms as "extremely suspect". Both of them resolve to the same site, which is an extremely sketchy domain parking company called Skenzo.

## Pricing options

Listed are comparative yearly renewal/transfer pricing from a few of the big providers. Note that these are the base 123 Reg prices and from what I understand from Steve they're asking for more than this for this renewal. All prices exclude VAT.

| Top-level domain | 123 Reg | Amazon Route53 | GoDaddy | Google | Cloudflare |
| --- | --- | --- | --- | --- | --- |
| .guide | $41.40 | $29.00 | $43.32 | $40.00 | $21.50 |
| .delivery | $69.00 | $47.00 | $70.41 | $60.00 | ??? |
| .games | $27.60 | N/A | $30.28 | $20.00 | ??? |
| .systems | $30.36 |$21.00 |$30.28 | $20.00 |??? |
| .pictures | $16.56 |$10.00 | $15.22 | $11.00 | ??? |
| .world | $41.40 | $30.00 | $42.31 | $40.00 | ??? |
| .games | $27.60 | $0.00 | $30.28 | $20.00 | ??? |

Amazon Route53 has the cheapest published prices, with the caveat that they do not currently offer a `.games` top-level domain. They are also highly unlikely to engage in the kind of unsavoury price-gouging and domain-parking that 123 Reg currently are.

However, Cloudflare are a special case: they offer what is essentially the wholesale price for registering/renewing a domain -- their price for `.guide`, which we know because we have `opmodel.guide` hooked up to Cloudflare, is cheaper than even Amazon. The catch is that their pricing isn't public thanks to agreements with the wholesalers, so Cloudflare needs to know about a domain before it'll tell you how much it would cost to register with them.

## Actions

Transfer all domains away from 123 Reg to either Route53 or Cloudflare as they come up for renewal. 

**Amazon Route53** is my preferred option since we're already in the AWS ecosystem and [we know their whole price list](https://d32ze2gidvkk54.cloudfront.net/Amazon_Route_53_Domain_Registration_Pricing_20140731.pdf). We would have to give up the `systemsthinking.games` domain but since we're not currently using it this is not a huge loss.

`transformation.delivery`, `rich.pictures`, `mapping.systems` and `opmodel.guide` all need to be transferred before they expire mid-November. As the transfer can take up to seven days, we should aim to start the process no later than the end of October. 

We also need to save and migrate the DNS settings from 123 Reg over to Route53 as well -- in the case of `opmodel.guide` we should think about pulling the DNS for it out of Cloudflare and into Route53, as Route53 is just as capable in terms of routing (less so in terms of DDOS protection).

We should also fix the `systemic.world` DNS so that it doesn't take you to a dodgy domain-parking site.
