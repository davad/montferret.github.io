---
title: "Ferret v0.10.0"
subtitle: "Comeback"
draft: false
author: "Tim Voronov"
authorLink: "https://www.twitter.com/ziflex"
date: "2020-02-21"
---

Hello fellow miners, **[Ferret v0.10.0](https://github.com/MontFerret/ferret/releases/tag/v0.10.0)** has been **finally** released!

It's been long and busy 6 months since the last release, and many of you might thought that the project got abandoned.    
We assure you it's not! But sometimes life happens and slows you down.

But we are back and ready to rock!

This release has many great features and improvements, thank to all our contributors.    
Let's take a look at the most important ones. You can find the full changelog [here](https://github.com/MontFerret/ferret/blob/master/CHANGELOG.md#0100).

# What's added
## I/O functions
You asked - we delivered! I/O functions are finally here.
In the beginning of the project, we were concerned about potential security threats access to filesystem could create, but seeing how in-demand this functionality is, we agreed to add it.    
Now ferret has new I/O namespaces with the following functions:

- ``IO::FS::READ``
- ``IO::FS::WRITE``
- ``IO::NET::HTTP:GET``
- ``IO::NET::HTTP:POST``
- ``IO::NET::HTTP:PUT``
- ``IO::NET::HTTP:DELETE``
- ``IO::NET::HTTP:REQUEST``

Enjoy!

## Loading HTML into memory
Another popular feature request - the possibility to load a prefetched HTML document into Ferret.    
With this release, you can finally do it with ``HTML_PARSE`` function.

{{< code fql >}}
LET file = IO::FS::READ(@myfile)

LET doc = HTML_PARSE(TO_STRING(file), {
    driver: 'cdp' // or 'http'
})

RETURN INNER_TEXT(doc, 'title')
{{</ code >}}

## Parameter value availability check
In this release, we do check pre-runtime to make sure all parameters are provided values in a script.    
It was very frustrating when your script was failing in the middle of the execution just because you forgot to provide a value for one of the paramaters.

# What's changed
## Case insensitivity
Finally! FQL keywords are case insensitive!

That's how Google Search query looks like in lower case:

{{< code fql >}}
let google = document("https://www.google.com/", {
    driver: "cdp",
})

input(google, 'input[name="q"]', "ferret")
click(google, 'input[name="btnK"]')

wait_navigation(google)

for result in elements(google, '.g')
    filter trim(result.attributes.class) == 'g'
    return {
        title: inner_text(result, 'h3'),
        description: inner_text(result, '.st'),
        url: inner_text(result, 'cite')
    }
{{</ code >}}

## Improved CDP driver performance
CDP driver performance has been drastically improved that brings CPU usage down from 60% for 3 pages to <1%.   

# What's fixed
## Redirects
For a long time, redirects that occurred during page navigation led to timeouts or corrupt page state.    
Now it has finally been fixed!   
```WAIT_NAVIGATION``` function has 2nd optional parameter, an object with the following fields:

{{< code go >}}
type Parameters struct {
    TargetURL string
    Timeout time.Duration
}
{{</ code >}}

``TargetURL`` is a regexp string that can be used to give Ferret a hint about what the destination url is:

{{< code fql >}}
LET doc = DOCUMENT("http://waos.ovh/redirect.html", {
    driver: 'cdp',
    viewport: {
        width: 1920,
        height: 1080
    }
})

CLICK(doc, '.click')

WAIT_NAVIGATION(doc, { targetURL: "redirect2\.html" })

RETURN ELEMENT(doc, '.title')
{{</ code >}}