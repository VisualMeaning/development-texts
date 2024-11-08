Extendscript is a forked, non-standard version of the Javascript language from the ES3 vintage.
It is used as a scripting language within Adobe Illustrator, and other products like Photoshop and After Effects.
In some places it's been deprecated and replaced by other languages (including other Javascript engines) but it's still the best or only option for certain extensions.

## Specs?

Adobe did previously have quite a bit of documentation around at least functions and apis of their products, but it's generally bit rotted and fallen off the internet.
So, the best resources for knowing how extendscript and Adobe extensions should work are:

* The independent [docsforadobe] github project
* Random blog entries and the old Adobe support forums to the extend search engines still have them in the index
* Experimentation (by trying to run things under the scripting engine of a current product)

In the interim the [ECMAScript version history] has marched onwards (after a long break in the middle) but no updates have happened to extendscript.


## Syntax differences

### Regexp literals

- `/` must be escaped in character classes.
- '=' can't appear at the start of expressions.
- Various newer regexp features like unicode are unsupported.
- More?

There are other [broken regexp] problems including at evaluation time.

### Comparisons

Comparing objects is enough of a mess in js, and extendscript gets some of them wrong, so using for instance [== null] may have extra-counter-intuitive behaviour.

### Non-standard and deprecated features

While `const` has been a reserved word since the early days of js, it didn't get semantics assigned till ES6. Extendscript supports `const` but not with ES6 semantics (and has no matching `let`) so it's best ignored and sticking to `var` and the old odd scoping rules.

Extendscript supports the (deprecated) [`with` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/with) that affects scope resolution. I don't know if extendscript correctly implements the semantics, it's best to just not use this feature.

`#strict on` is not `"use strict";`. Use neither.

### Works but weird

Authors of extendscript also forked off from the rest of humanity in their thinking about what good js looks like. So, there may be confusing elements in existing scripts around code design, layout, formatting, and naming that are perfectly valid constructs but not easy to follow.


## Imports

When extendscript was made js did not have any of the several iterations of support for breaking up code across multiple files (amd, commonjs, ES6 modules), so it has a different way that does not use `import` as a keyword or function.
In essence, there is an include preprocessor directive, and everything just gets run in the same scope.

There are two supported spellings for preprocessor directives:

```
#include "json2.js"
//@include "json2.js"
```

The comment-at version should be used as that does not clash with the supported syntax of ES3 so a standard parser can still be used (but will be ignorant of the directive).


## Tooling

### Dollar

There is a global [$ object](https://extendscript.docsforadobe.dev/extendscript-tools-features/dollar-object.html) that exposes some engine functionality and a `$.write` as an analog to `console.log` for debug printing.

It's not like the JQuery dollar object.

### Debugger

Formerly there was an 'Extendscript Toolkit' but that's deprecating in favour of a VSCode extension.

> "It’s semi-buggy, generally unreliable, and seems to be meant for self-contained instant-run scripts (as opposed to ScriptUI panels), but it can be helpful in some cases."

### Linting

The [`eslint-plugin-extendscript`](https://www.npmjs.com/package/eslint-plugin-extendscript) package can be used to make eslint mostly useful for catching syntax and other errors.

For checking imports and exports are as intended, these can be declared in comments at the top of each file.


[ECMAScript version history]: https://en.wikipedia.org/wiki/ECMAScript_version_history
[docsforadobe]: https://github.com/docsforadobe/
[extendscript overview]: https://extendscript.docsforadobe.dev/introduction/extendscript-overview.html
[broken regexp]: https://indiscripts.com/post/2022/01/the-hard-Problem-of-quantified-alternatives-extendscript
[== null]: https://indiscripts.com/post/2020/04/why-you-should-never-use-x-equals-null-in-extendscript
