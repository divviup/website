---
author: Bran Pitman
date: 2023-10-11T00:00:00Z
slug: divvi-up-live-in-firefox
title: "Divvi Up is providing privacy-preserving metrics for Firefox"
excerpt: "Firefox telemetry metrics will be collected in a privacy-preserving way with Divvi Up."
---

We've made tremendous progress on developing Divvi Up and the Internet Engineering Task Force (IETF)'s Distributed Aggregation Protocol [(DAP)](https://www.ietf.org/archive/id/draft-ietf-ppm-dap-02.html) standard. Today, I'm excited to share that a deployment of DAP is in the works for [Mozilla's Firefox browser](https://www.mozilla.org/en-US/firefox/new/).

The idea behind Divvi Up is to resolve much of the tension between wanting to know information about a population of people and achieving that by collecting information about individuals that might compromise their privacy. Firefox, as a browser that keeps its users' privacy at the forefront, was the ideal partner to roll out the first deployment of Divvi Up.

Throughout 2022 and 2023, we collaborated with Firefox engineers, along with contributors from many other organizations, to develop the DAP specification in the IETF. Simultaneously, Divvi Up engineers developed a DAP client, collector, and aggregator called [Janus](https://github.com/divviup/janus) (we named it after the Roman deity of duality!).

Initial pilots with Firefox proved the privacy guarantees and other properties of the system. Various metric types based on the [Prio aggregation scheme](https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/corrigan-gibbs) by Dan Boneh and Henry Corrigan-Gibbs have been collected and verified as correctly aggregated in a test environment, showing that numerical metrics can be aggregated in a privacy-preserving way. The initial performance properties of the system gave us the confidence to move forward with live user metrics.

The Firefox deployment is a good demonstration of how DAP can help. Collecting user metrics such as the relative frequency of visits to different web pages would help Firefox engineers identify the most important bugs to fix and features to optimize, but this information could also be privacy-invasive if it wasn't de-identified and aggregated. In the DAP system, metrics are split into two unintelligible parts. One goes to a Divvi Up aggregator and the other goes to an aggregator run by Mozilla. Each organization aggregates the parts received from all clients and returns a partial sum to Mozilla, who then combines the results into one aggregate output.

![](/images/blog/Divvi-Up-How-It-Works-Mozilla1.mov)
<video poster="/video/Divvi-Up-How-It-Works-Mozilla1-First-Frame.jpg" style="width: 100%; aspect-ratio: 16/9;" autoplay="autoplay" playsinline="" loop="" controls="" muted="">
              <source src="/video//video/Divvi-Up-How-It-Works-Mozilla1.mov" type="video/mp4">
              Your browser does not support the video tag.
            </video>

We are excited to continue developing DAP and Janus in the future. We will soon begin testing with the [Poplar aggregation scheme](https://eprint.iacr.org/2021/017.pdf), which will allow commonly reported values like URLs to be discovered without impinging on the privacy of individual reports.

To learn more about Mozilla's deployment, check out their announcement [blog post](https://blog.mozilla.org/en/products/firefox/partnership-ohttp-prio/).

Divvi Up is a project of the nonprofit [Internet Security Research Group](https://abetterinternet.org/) (ISRG), the organization behind [Let's Encrypt](https://letsencrypt.org/) and [Prossimo](https://www.memorysafety.org/). If you or your organization are interested in helping advance the work of ISRG via funding, advocacy, or another idea, please get in touch via sponsor@abetterinternet.org.