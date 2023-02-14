---
author: Tim Geoghegan
date: 2023-02-14T12:00:00Z
slug: DAP-Update
title: "An update on Divvi Up and the Distributed Aggregation Protocol"
excerpt: "A new aggregation algorithm and preparing for production."
---

In 2023, we aim to take Divvi Up, our service for private statistical aggregation of software telemetry, into production. Divvi Up implements the [Distributed Aggregation Protocol](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/) (DAP), an open standard we're helping develop through the Internet Engineering Task Force (IETF), so a lot of our work goes hand in hand with improvements to the underlying standards.

In this post, we're going to discuss some of the changes we've been working on that are targeted for the next draft of DAP and should form the basis of the Divvi Up subscriber-facing deployments later this year. In particular, we're going to look at two broad themes: the Poplar1 algorithm and robustness of the protocol.

Poplar1
-------

DAP is designed as a framework that can run a variety of different private aggregation algorithms. Thus far, Divvi Up has focused on running the Prio3 family of algorithms, which is well suited to aggregating over numerical measurements (e.g., the number of milliseconds an operation took when performed repeatedly, or the number of times users have pressed a button per day). We've also been developing another algorithm called Poplar1, which is specialized for aggregating over strings. For example, a browser vendor could use this to learn what the most frequently visited URLs are without learning any individual's highly sensitive and private browsing history.

The upcoming draft of DAP includes numerous changes that improve the performance of Poplar1 and harden it against potential attacks by refining its use of pseudorandom number generators. David Cook from the Divvi Up team has been working on a [fast implementation](https://github.com/divviup/libprio-rs) in Divvi Up's open-source implementation of Prio3 and Poplar1.

Lots more analysis and performance testing is needed, but we're optimistic that Poplar1 will scale well and will enable application developers to gain new insights into how their software is used, while respecting user privacy.

Robustness
----------

As we move from last year's experimental deployments, we need to make sure that the protocols we are building on are sufficiently robust to handle real customer data.

On the one hand, that means applying rigorous security analysis to the cryptographic primitives so that we can ensure that our higher level protocols are using them safely. Just this month, researchers at Cloudflare, Google, Oregon State University and the University of California San Diego published a [paper](https://eprint.iacr.org/2023/130.pdf) that does just that. Our colleague and DAP co-editor Chris Patton has written a [concise summary of the impacts of this analysis on the protocol on the IETF's Crypto Forum mailing list](https://mailarchive.ietf.org/arch/msg/cfrg/-errBjFRvCqi7KuAxoZwH6iY4Vc/). We'll be working over the coming weeks to amend the protocol as needed so that we can be confident our users get all the privacy protections that these algorithms can offer.

Besides cryptographic robustness, we also have been working to make sure that DAP will be operable at the large scale that we hope Divvi Up will reach. In particular, we've been reviewing DAP's use of HTTP to ensure that the requests it describes are idempotent. In this context, idempotence means that nothing bad happens should a request get sent twice. That's important because despite our best efforts, the Internet is not a perfectly reliable medium of transmission, and so any protocol that aims to be reliable needs to consider the possibility of transient network failures and include suitable affordances for retrying when failures are encountered.

Conclusion
----------

DAP still has a long way to go on its journey towards becoming an IETF standard, but this work brings it closer to being deployable at Internet scale and broadens the use cases it can address. The Divvi Up team will continue to develop DAP and our implementation so they will reach wider audiences this year.

About Divvi Up
--------------

Divvi Up is a project of the nonprofit [Internet Security Research Group](https://abetterinternet.org/) (ISRG), the organization behind [Let's Encrypt](https://letsencrypt.org/) and [Prossimo](https://www.memorysafety.org/). If you or your organization are interested in helping advance the work of ISRG via funding, advocacy, or another idea, please get in touch via sponsor@abetterinternet.org.