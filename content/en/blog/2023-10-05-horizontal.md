---
author: Dan Fernelius and David Cook
date: 2023-10-05T00:00:00Z
slug: horizontal
title: "Bringing Privacy-respecting Telemetry to Human Rights Defenders"
excerpt: "We’re pleased to announce that Horizontal is Divvi Up’s first production user, bringing privacy-respecting telemetry to their work at the intersection of technology, human rights, and justice."
images:
  - /images/blog/divvi-up-how-it-works.png
---

Today we're pleased to announce that Horizontal is Divvi Up's first production user, bringing privacy-respecting telemetry to their work at the intersection of technology, human rights, and justice. [Horizontal](https://wearehorizontal.org/) is a nonprofit organization supporting frontline defenders, activists, and journalists through digital security and tool development.

## Advancing Internet Freedom

As Horizontal says, "[W]hile the Internet has connected us, amplified the voices of the most vulnerable, and helped build movements, it has also become a tool to survey, monitor, and repress." What's more, even for organizations seeking to protect and advance user privacy and security such as ISRG and Horizontal, technologically doing so can be difficult, if not impossible.

Such was the case for Horizontal's application Tella, a tool that enables users to safely and securely collect and store information, making it easier and safer to document human rights violations and collect data. While the team would have benefited from telemetry of the app's performance, there previously wasn't an easy method to collect telemetry without accessing some sort of user information. As Horizontal's Founder and current programs lead, Raphael Mimoun, said in a recent conversation, when it comes to user data "if it exists, it's a risk."

## The possibility of privacy-respecting telemetry

Horizontal and ISRG first connected via the [Open Technology Fund](https://www.opentech.fund/), a respected nonprofit organization advancing Internet freedom worldwide, in early 2023. Mimoun was interested in exploring how Divvi Up could allow Horizontal to finally gain insights into their services via privacy-preserving telemetry. "We launched Tella four years ago. We have zero idea of how people are using it except for what we've heard qualitatively from our community," said Mimoun.

For any organization, storing user data is a privacy and compliance risk. For organizations like Horizontal, it's an existential threat to the organization and its users, and out of line with their values. Mimoun continued, "We have reached a point with Tella where it is mature. We know we need to step up our game and learn about how people are using it."

## Technologically protecting privacy

Divvi Up makes it possible for Horizontal to gain insights about how their population of users interacts with Tella and Shira without any knowledge of specific users. Unlike any other privacy-respecting system available, Divvi Up has the math and design to protect user privacy while delivering telemetric insights. Here's a deep dive on how it works:

When an app contributes to telemetry, it first turns the measurement of interest into one or more numbers. When using Divvi Up, each number is split up into two shares using an additive secret sharing scheme. One of the secret shares is a randomly chosen number, and the other is computed by subtracting the same random number from the measurement number. (All addition and subtraction is done using modular arithmetic, so negative numbers wrap around to a different positive number.) Now, the two secret shares add up to the original measurement, but each secret share on its own looks like a random number. Each set of secret shares will be uploaded to separate servers, one operated by Divvi Up, and the other by Horizontal. The secret shares won't ever be reunited once they leave the app, which protects the privacy of individual users' measurements.

![](/images/blog/divvi-up-how-it-works.png)

The two servers cooperate to compute aggregate statistics from all the sets of secret shares they receive. Multiple uploaded secret shares from different users are grouped together into a batch. Then, each server adds together its secret shares from those messages. Since addition can be rearranged freely, this sum of secret shares of measurements is equal to one secret share of the sum of the measurements. The two servers provide these intermediate sums to Horizontal, who then adds them together, getting the sum of measurements from all uploaded data in that batch. This provides them the telemetry they need for insights into how their apps are being used, so they can make informed decisions about development priorities, while still protecting the privacy of their users.

There's one more component beyond the scope of this explanation: clients upload proofs along with their secret shares, specifically "zero knowledge proofs on secret shared data". This concept has been developed through a series of academic papers, starting with [the original Prio paper](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-corrigan-gibbs.pdf), through [Davis, et. al., 2023](https://eprint.iacr.org/2023/130). These proofs allow the servers to verify that the secret shares they hold came from a measurement that satisfies some validity constraint, for example, that it's either zero or one, without learning anything more about the other server's secret share or the original measurement. The two servers verify these proofs before adding the corresponding secret shares to their partial sums. This prevents malfunctioning devices or maliciously constructed messages from having an outsize effect on telemetry results with a small number of requests.

Through the combination of simple addition and validity conditions checked through zero knowledge proofs, we can process multiple different kinds of metrics:

* Count: Measurements are true or false values. The results tell how many measurements were true out of the total number of valid measurements.
* Sum: Measurements are integers in some predefined range. The result is the sum of all measurements. This is useful for computing an average of a continuous quantity.
* Sum of Vectors: Measurements are vectors of integers, each in some predefined range. The result is the element-wise sum of all measurements. This can process multiple counters or sums together.
* Histogram: Each measurement increments one of several counters. The result is a vector of counts. This can be used to estimate the distribution of a continuous variable, to count the occurrences of values of a categorical variable, or to estimate the joint distribution of multiple variables with a multidimensional histogram.

## Getting started with Divvi Up

Horizontal is a team of ten located in seven countries around the world. Juan Dans, software engineer, set up Divvi Up in close collaboration with the Divvi Up team. "There was a process of learning the tool. At first, I approached it as a product I could implement but I realized it is something that was still really hands-on developing, so it needed to be a hands-on collaborative process," said Juan. "At the end, we had two teams working together in the same way." This collaboration led to learnings for both parties; learnings we've [taken forward ](https://divviup.org/blog/preparing-for-first-subscribers/)for both Divvi Up and the development of the IETF standard, [Distributed Aggregation Protocol (DAP)](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/).

We at ISRG are thrilled Horizontal is among the first users of Divvi Up on what we hope is a journey to wide-scale adoption of privacy-preserving metrics. "It was extremely exciting to know that we are working on something totally new, and working with an organization like the one who has a track record of building great things like Let's Encrypt," said Raph.

Divvi Up will continue to onboard our first subscribers over the next few months. If you'd like to bring privacy-respecting telemetry to your organization, the [Divvi Up waitlist ](https://divviup.org/get-involved/)is now open. If you'd like to chat further, we'd be happy to hear from you via our contact information below.

We'd like to thank Horizontal for their collaboration and enthusiasm for using this service for such a noble and necessary purpose. We'd also like to thank the Open Technology Fund for their financial support of both Horizontal and Divvi Up.

Divvi Up is a project of the nonprofit Internet Security Research Group (ISRG), the organization behind Let's Encrypt and Prossimo. If you or your organization are interested in helping advance the work of ISRG via funding, advocacy, or another idea, please get in touch via <sponsor@abetterinternet.org>.

Divvi Up waitlist: https://divviup.org/get-involved/ 

Get in touch: Dan Fernelius, dan@abetterinternet.org