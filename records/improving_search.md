# Improving search

Currently platform search simply iterates over certain text fields in the model and determines matches using regular expressions. This is sufficently fast, but lacks some of the features possible with other approaches.

Possible wishlist:
* good tokenisation
  * proper handling of punctuation and word boundaries
  * removing markdown syntax
  * language aware tokenising
* good processing
  * normalisation
  * language aware processing
  * stemming
  * even fancier things eg. based on pronunciation
* result ranking
* fuzzy matching
* search suggestions
* output of match locations
* tailored behaviour for different fields
* async (and perhaps cancellable) queries

The best way to improve functionality is to adopt a library that supports some or all of these. Such libraries will have some way of building one or more indexes from documents, and then running search queries using that data.


## Where does the work get done

There's also a choice in where the work to perform search operations happens. It can be:

* API to server-side search query interface
* Client-side query against server pre-built index
* Client-side query against index built on load

Given we load the complete model client-side, the main benefit of a fully server-side engine (being able to have huge indexes) is not something we're bothered about. We might at some point be interested in server-side or offline index building though.


## Libraries

There are a wealth of options to pick from, so what are the criteria?

Basically just want a library with the functionality we need, and reasonable ongoing maintenance. Several of the projects are by people who have a consultancy business attached to their free software, which is likely a positive indicator for longevity.

Most of the javascript libraries advertise "zero dependencies" as a positive, this means they are either re-implementing or avoiding complex data structures for their indexes. 

Both the external interface we'll be using, and the internal code quality are of interest.

### [Sonic]

* Server-side
* Language: Rust
* Last release: [2 months ago 1.4.8](https://crates.io/crates/sonic-server)
* Maintainer's photo: Pull-focus face a bit edgily close

Looks like a cool (modern) alternative to elasticsearch, if we wanted to go down the route of running dedicated server-side infrastructure for search.

### [meilisearch]

* Server-side
* Language: Rust
* Last release: it complicated but [two weeks ago 1.6.2](https://github.com/meilisearch/meilisearch/releases)
* Maintainer's photo: Looks down on you, and in the photo

Has client-side api emphasis as they want you using their cloud service but can also self-host. A lot of functionality including for geographic data, multiple languages, and adding some AI stuff with embeddings.

### [Stork]

* Client-side with cli index builder
* Language: Rust
* Last release: [a year ago 1.6.0](https://crates.io/crates/stork-lib)
* Maintainer's photo: Outside in the sun holding a hoop

Looks like a cool approach to offline index building and browser-side querying. The creator announced in 2023 he is [winding down work on Stork] though is open to others adopting the code. Probably more useful to learn lessons from than try to use directly.

### [Lunr]

* Client-side
* Language: Javascript
* Last release: [three years ago 2.3.9](https://www.npmjs.com/package/lunr)
* Maintainer's photo: Upscaled jpg black and white with a collar 

"A bit like Solr, but much smaller and not as bright." is a pretty funny intro. It's not the bloat of an Apache project implemented in Java - but of the options it's one of the larger and more fully featured!

Probably a bit too dead to use unless we want to start maintaining it.

### [Fuse]

* Client-side
* Language: Javascript
* Last release: [four months ago 7.0.0](https://www.npmjs.com/package/fuse.js)
* Maintainer's photo: Evokes a hyperactive video game detective avatar

Seems to be a solid library with a bunch of the most commonly needed options like fuzzy matching and ranking. Has boolean cominators and some special syntax for asking for an exact match. Most of the smarts seem to be on the query side, there's nothing at parse time for diacritic handling with normalised but preserved matching.

### [orama]

* Client-side
* Language: Javascript
* Last release: [today 2.0.6](https://www.npmjs.com/package/@orama/orama)
* Maintainer's photo: Threatens to push you over and steal your skis

Main advantage seems to be it's built on vectors natively so does that well. Not really what we need currently. Does also have geo searching which is neat, and some solid relevance metric with field boosting for customisation.

### [flexsearch]

* Client-side
* Language: Javascript
* Last release: [a month ago 0.7.43](https://www.npmjs.com/package/flexsearch)
* Maintainer's photo: Upscaled crop from a holiday photo with someone else's shoulder

Quite a large external interface, but does seem to include sensible options at index creation that aren't obviously available in other options. Good readme with lots of detail. The benchmarks link is dead. Eyeball code quality level looks a little lower, ancient idiosyncrasies like `prototype` usage. Actively maintained with and the [flexsearch 0.8 discussion] is encouraging. Several things missing currently may be coming in the next release: intermediate results with scoring rather than just ranking; reporting of what exactly matched for highlighting.

### [tinysearch]

* Client-side
* Language: Rust
* Last release: [seven months ago 0.8.2](https://crates.io/crates/tinysearch)
* Maintainer's photo: Professionally shot to convey fun and interesting

Run Rust in the browser using WASM! There's a neat [tinysearch blog post] about implementing this. I've met the author at a previous FOSDEM and think he's solid.

### [minisearch]

* Client-side
* Language: Javascript
* Last release: [three months ago 6.3.0](https://www.npmjs.com/package/minisearch)
* Maintainer's photo: Hanging in the park but only into the photographer

Skips bundling a few options like fancy tokenising and processing but provides an interface for you to bring your own. Eyeball code quality looks pretty good, sane looking typescript and lots of inline documentation.


## Tiebreakers

Doing a giant feature matrix would be a bit of a pain, so this is more an impression of what support looks like at a glance from the docs and code for a few key things.

* build indexes with (or document with fields with) different parameters
  * lunr
  * orama (document-based)
  * flexsearch
  * minisearch (document-based, mostly on query)
* support en, zh, ru, (fr, pl) languages reasonably
  * lunr (but maybe not, claims support for 14 languages but where's the code?)
  * orama (bundles quite a bit of support but incomplete because of higher ambitions)
  * flexsearch (builtin somewhat stubby config for different scripts)
  * minisearch (provides interfaces to bring your own tokenization and term processing)
* match with and without accent for a name with an exact match ranking higher
  * flexsearch (via advanced index option)
* use selective prefix match for as-you-type results of some form
  * lunr (very customizable at the query term level)
  * flexsearch (prefix can be set per index or document field)
  * minisearch (prefix option global to query, also has suggestions feature)


## Recommendation

We should try using both flexsearch with various options, and minisearch with some specific transforms we implement.



[Sonic]: https://journal.valeriansaliou.name/announcing-sonic-a-super-light-alternative-to-elasticsearch/
[meilisearch]: https://www.meilisearch.com/
[Stork]: https://github.com/jameslittle230/stork
[winding down work on Stork]: https://jameslittle.me/blog/2023/winding-down-stork/

[tinysearch]: https://github.com/tinysearch/tinysearch
[tinysearch blog post]: https://endler.dev/2019/tinysearch/

[Lunr]: https://lunrjs.com/
[Fuse]: https://www.fusejs.io/
[flexsearch]: https://github.com/nextapps-de/flexsearch
[flexsearch 0.8 discussion]: https://github.com/nextapps-de/flexsearch/discussions/415
[orama]: https://docs.oramasearch.com/
[minisearch]: https://lucaong.github.io/minisearch/

[IndexedDB]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
