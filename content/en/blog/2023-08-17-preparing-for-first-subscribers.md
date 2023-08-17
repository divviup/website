---
author: Josh Aas
date: 2023-08-17T00:00:00Z
slug: preparing-for-first-subscribers
title: "Preparing for Our First Subscribers & New Funding"
excerpt: "On the road to a privacy-preserving metrics MPC service that is ready for production"
---

We at ISRG have been building Divvi Up, a privacy-preserving metrics system that uses a novel multi-party computation system, since mid-2021. Since we started, we've shepherded five drafts of the [Distributed Aggregation Protocol](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/) (DAP) through the Internet Engineering Task Force (IETF) alongside the development of our own implementations of that protocol. We're creating a service that will reduce the unique information application users expose about themselves and their behaviors by turning individual data points into aggregate, anonymized metrics.

## Protocol Development

Divvi Up has been [running](https://github.com/divviup/janus) DAP-02 and DAP-04 for some time now. We are looking to land on DAP-05 shortly as a long-running version upon which we will build our production service. The biggest difference between DAP-04 and DAP-05 is an improvement in the performance of the aggregation subprotocol thanks to an approximately [50%](https://datatracker.ietf.org/meeting/117/materials/slides-117-ppm-tgeoghegan-ietf-117-dap-updates-00#page=2)  [reduction](https://datatracker.ietf.org/meeting/116/materials/slides-116-ppm-dap-allowing-more-than-one-helper-or-not-00.pdf) of network requests. This efficiency gain will decrease the cost of running Divvi Up and improve our ability to scale. We are collaborating with initial partners to ensure there is an ecosystem of providers who are all interoperable on DAP-05.

## BYO Helper

We want to make it as easy as possible for application developers to get aggregate, anonymized metrics through Divvi Up. Our vision for the service is to make Divvi Up a one-stop-shop where we will operate as the primary aggregation server and take care of finding a secondary "helper" server for the subscriber. However, we know that some subscribers will want to choose their own helper server, sometimes even filling that role themselves. We've been working on enabling a [Bring Your Own (BYO) helper](https://github.com/divviup/janus/issues/1486) option in Divvi Up to enable this flexibility. Subscribers can use our implementation of DAP, called [Janus](https://github.com/divviup/janus), which comes with the necessary control plane interfaces to make this a possibility out of the gate.

## Subscriber Onboarding

One of the key elements of a successful self-serve service is the ability to spin up new metrics collection without too much time investment or the involvement of a Real Live Human. The Divvi Up management console is under construction to achieve just that. We've sketched out a flow for new subscribers to determine what metrics they want to collect and over what period of time. Over the coming months, we will continue to develop the management console so it can help subscribers get up and running quickly, and then enable subscribers to efficiently get metrics that can provide valuable insight to inform application design and functionality decisions. If you're interested in being one of the first to test out the system, join our [waitlist](https://divviup.org/get-involved#waitlist).

## New Funding

We've invested over $2M of funding from contributors like [Ford Foundation, Robert Wood Johnson Foundation, Open Technology Fund](https://divviup.org/blog/2022-06-28-announcing-divviup-funding/), Bill & Melinda Gates Foundation, [Internet Society Foundation, and Meta](https://divviup.org/blog/divvi-up-funding-update/). We're happy to announce today that Omidyar Network is joining that leading group with a $100,000 grant.

<img style="width: 100%; margin: 60px auto 80px; max-width: 715px; display: block;" class="img-flu id" src="/images/blog/Omidyar_Network_Logo.png" alt="Omidyar Network Logo" />

## About Divvi Up

Divvi Up is a project of the nonprofit [Internet Security Research Group](https://abetterinternet.org/) (ISRG), the organization behind [Let's Encrypt](https://letsencrypt.org/) and [Prossimo](https://www.memorysafety.org/). If you or your organization are interested in helping advance the work of ISRG via funding, advocacy, or another idea, please get in touch via sponsor@abetterinternet.org.