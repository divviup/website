---
author: David Cook
date: 2024-08-07T00:00:00Z
slug: combining-privacy-preserving-telemetry-with-differential-privacy
title: "Combining Privacy Preserving Telemetry with Differential Privacy"
excerpt: "An overview of the how differential privacy can be combined with the Distributed Aggregation Protocol, and related recent developments."
images:
- /images/blog/2024-08-08-Social-Share.png
---


Divvi Up is a privacy preserving telemetry service. The privacy protections are achieved by ensuring no individual piece of telemetric data is revealed to the person or organization collecting the data -- only the summary or aggregate data. But there are further steps we can take to ensure privacy, in order to limit how much the aggregate data itself reveals about the raw data it was computed from. In this post, we'll introduce the concept of differential privacy, a robust and well-understood privacy framework which can do just that. We will describe how differential privacy can be combined with the Distributed Aggregation Protocol (DAP), which Divvi Up is built on, and the advantages we get by combining them rather than using either DAP or differential privacy on their own. We'll introduce new support for differential privacy mechanisms in Janus, the DAP aggregator powering Divvi Up. We'll also go over some recent developments from others in the DAP ecosystem, relevant to the intersection of differential privacy and DAP.

*Are you new to all of these terms? You might want to pause here and try out our [hands on tutorials and technology overviews](https://docs.divviup.org/) and continue reading once you are done.*

### Background: Differential Privacy

Differential privacy is a mathematically rigorous definition of privacy, first introduced in a [2006 paper by Dwork et. al.](https://people.csail.mit.edu/asmith/PS/sensitivity-tcc-final.pdf) Briefly, a process is differentially private if the statistical distribution of its output is always close to the statistical distribution of its output on the data set with one measurement added, removed, or changed. This means that it is hard for an attacker to learn anything about any individual's measurement, or even tell if an individual's measurement was included at all. There are a few related definitions of differential privacy that use different concepts of statistical closeness, or different methods of modifying the data set, but they all share this form.

Unlike prior approaches (such as de-identification), the guarantees that differential privacy provides still hold even if the attacker has access to auxiliary information related to the data set being protected. It is also possible to compose multiple differentially private analyses on the same data set, and still get meaningful privacy guarantees. Differential privacy is commonly achieved by adding carefully calibrated noise from a particular probability distribution, though there are alternative mechanisms as well.

Since its inception, the field of differential privacy has produced a long line of research results and successful applications. Famously, the U.S. Census Bureau [used differential privacy in the 2020 Census](https://www.census.gov/programs-surveys/decennial-census/decade/2020/planning-management/process/disclosure-avoidance/differential-privacy.html). To learn more about the foundations and applications of differential privacy, we recommend [this primer from the Harvard Privacy Tools Project](https://privacytools.seas.harvard.edu/files/privacytools/files/pedagogical-document-dp_new.pdf).

### Differential Privacy in Secure Aggregation Protocols

Combining a secure aggregation protocol such as DAP with differential privacy techniques has some great benefits. First of all, while DAP keeps individual measurements private, it reveals the aggregate result to the collector by design. Depending on the application and the privacy goals, the aggregate result itself could reveal more information than desired. This is exacerbated with high-dimensional data. For example, if the aggregate result is a histogram, and one histogram bucket is selected very rarely, we may not want to reveal whether it showed up once or not at all. Adding a differential privacy mechanism resolves this by adding noise to all the bucket counts, thus limiting the amount of information that is revealed by the aggregate results about rare measurements.

DAP also allows us to implement a differential privacy mechanism in the same distributed trust paradigm as the aggregation process itself. Traditional applications of differential privacy assume that there is a trusted database curator that keeps the entire data set private, and the curator only releases differentially private query results to other parties. For example, the U.S. Census Bureau holds raw survey results and goes to great lengths to protect them, while only publishing summary data products with noise added. When using DAP, we can implement our differential privacy mechanism by having the non-colluding aggregators add noise to secret shared data. The individual measurements remain private as before, and now the collector gets a noised aggregate result when it adds its secret shares together.

We expect it will be common for mature DAP deployments to include some form of differential privacy mechanism. Differential privacy is a broadly-accepted framework, and the additional protections it provides address residual concerns like [aggregation function leakage](https://www.ietf.org/archive/id/draft-ietf-ppm-dap-11.html#section-7-8.4.1) and [Sybil attacks on privacy](https://www.ietf.org/archive/id/draft-ietf-ppm-dap-11.html#section-7.1-5.3.1).

### Janus Support for Differential Privacy

[Version 0.7.27 of Janus](https://github.com/divviup/janus/releases/tag/0.7.27), the DAP aggregator built by the Divvi Up team, introduced support for new differential privacy mechanisms. Previously, Janus included support for a DP mechanism used with a Prio3 VDAF for federated learning, contributed by [@MxmUrw](https://github.com/MxmUrw) and [@ooovi](https://github.com/ooovi). Version 0.7.27 adds two new DP mechanisms that are compatible with the Prio3Histogram and Prio3SumVec VDAFs. You can add these DP mechanisms when creating a task via the divviup CLI tool or the `divviup-client` Rust library. Here is an example of creating a 0-10 histogram with a DP strategy:

```
divviup task create --name net-promoter-score \
    --leader-aggregator-id $LEADER_ID --helper-aggregator-id $HELPER_ID \
    --collector-credential-id $COLLECTOR_CREDENTIAL_ID \
    --vdaf histogram --categorical-buckets 0,1,2,3,4,5,6,7,8,9,10 \
    --differential-privacy-strategy pure-dp-discrete-laplace \
    --differential-privacy-epsilon 1 \
    --min-batch-size 100 --max-batch-size 200 --time-precision 60sec
```

Each of these DP mechanisms follows a similar overall structure. Both aggregators independently add noise to their aggregate share, before sending it to the collector. This ensures that, so long as at least one aggregator is honest, the process remains differentially private. The magnitude of the noise is determined by the VDAF, its parameters, the differential privacy budget, and its parameters. The probability distribution of the noise also varies between mechanisms. We currently use the [discrete Gaussian and discrete Laplace distributions](https://arxiv.org/abs/2004.00010), depending on the scenario.

The "global sensitivity" of the VDAF's aggregation function encapsulates the VDAF's impact on noise magnitude. It measures the maximum influence that one measurement can have on the output of the aggregation function. Since Prio3 VDAFs already check that measurements meet specific validity criteria, measurement values are limited to specific ranges, which helps keep the sensitivity of the aggregation function low. This improves the privacy-utility tradeoff users face when choosing differential privacy budget parameters. For example, the global sensitivity of Prio3Histogram is very low, since each measurement can only increment a counter once. This turns out to be another great synergy between differential privacy and DAP.

Janus's support for differential privacy mechanisms is extensible, so we expect to continue adding more as use cases arise. We can also support multiple kinds of differential privacy mechanisms for the same VDAF, each satisfying a different variant of differential privacy, to support applications with different needs.

### &ldquo;Differential Privacy Mechanisms for DAP&rdquo;

The draft document [Differential Privacy Mechanisms for DAP](https://wangshan.github.io/draft-wang-ppm-differential-privacy/draft-wang-ppm-differential-privacy.html) describes the privacy goals, threat models, and available options when combining differential privacy with DAP. It discusses multiple differential privacy mechanisms, and combinations of mechanisms, wherein noise is added by DAP clients, DAP aggregators, or both. Two concrete differential privacy mechanisms are provided for histograms. In one, the client adds noise using a randomized response mechanism (flipping each histogram bucket in the measurement with some probability) and the [Prio3MultiHotCountVec VDAF](https://cfrg.github.io/draft-irtf-cfrg-vdaf/draft-irtf-cfrg-vdaf.html#name-prio3multihotcountvec) is used as-is to aggregate the results. In the other, aggregators add noise to aggregate shares, using the Prio3Histogram VDAF. This document is still under development, and we expect it will be a key resource for combining differential privacy and DAP, especially as implementers gain more operational experience.

### Generating Noise in MPC

Ben Case and Martin Thomson recently published a new draft document titled ["Simple and Efficient Binomial Protocols for Differential Privacy in MPC"](https://private-attribution.github.io/i-d/draft-case-ppm-binomial-dp.html), and presented it at the Privacy Preserving Measurement Working Group meeting at IETF 120. This proposes a differential privacy mechanism that uses noise from the [binomial distribution](https://en.wikipedia.org/wiki/Binomial_distribution), and samples that noise using multi-party computation, or MPC. As opposed to the above mechanisms where both aggregators add independent noise, this would allow adding noise only once, improving the privacy-utility tradeoff. This mechanism also fits in the distributed trust paradigm, because neither server knows what noise value was generated and added.

The binomial distribution is simple to compute, so it is a good fit for MPC where certain operations get more expensive. The two servers will run a coin flipping protocol many times, which generates secret shares of 0 or 1, and then add the secret shares together. The sum of those secret shares is our binomial sample. The servers will add their secret shares of the binomial sample to their aggregate shares before sending them to the collector. The collector will add up the aggregate shares, and then subtract out the mean of the binomial distribution, in order to get the noised and de-biased aggregate result.

With a large enough number of coin flips, the binomial distribution is a good approximation to the Gaussian distribution. Adding either Gaussian noise or binomial noise provides us "(ε, δ)-differential privacy", though in this case, with the binomial distribution, the formulas for ε and δ are a bit more complicated.

Integrating this into DAP may require some additional rounds of communication, depending on how the coin flipping protocol is instantiated. This could require a change to DAP, or an extension to it. While work on this mechanism is just getting started, the outlook is promising, because it will provide the optimal amount of noise in DAP's threat model.

### Conclusion

We're excited about the privacy benefits that differential privacy in DAP can bring. Differential privacy is a proven approach to handling private data. Building on top of DAP as a secure aggregation protocol is especially useful, because no one party has access to all the raw data, which is rare in differential privacy mechanisms.

If you have questions about using Divvi Up for your telemetry use case, please get in touch at <contact@divviup.org>. If you're interested in following the development of this technology we recommend starting with the [PPM Working Group mailing list](https://mailman3.ietf.org/mailman3/lists/ppm@ietf.org/), and the repositories where the individual Internet Draft documents referenced above are being developed. (See the repositories for "[Differential Privacy Mechanisms for DAP](https://github.com/wangshan/draft-wang-ppm-differential-privacy/)" and "[Simple and Efficient Binomial Protocols for Differential Privacy in MPC](https://github.com/private-attribution/i-d)")

*Want to learn more about Divvi Up and privacy respecting statistics collection? Get started with Divvi Up, in under 5 minutes, with Docker and your command line. Our new tutorial will have you divvi'ing user statistics in no time. <https://docs.divviup.org/command-line-tutorial> Once your local environment is set up, try adding `--differential-privacy-strategy pure-dp-discrete-laplace --differential-privacy-epsilon 1` to your task creation command.*