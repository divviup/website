---
author: Tim Geoghegan
date: 2024-07-25T00:00:00Z
slug: OHTTP-now-available
title: "Oblivious HTTP now available on Divvi Up"
excerpt: "A stronger privacy posture with OHTTP and DAP for telemetry."
---

We are pleased to announce that Divvi Up now includes an Oblivious HTTP (OHTTP) gateway, allowing Distributed Aggregation Protocol (DAP) clients to contribute measurements while minimizing what either aggregator learns about them.

Subscribers will need to arrange for a trusted party to run an OHTTP relay server, such as [Fastly](https://docs.fastly.com/products/oblivious-http-relay) or [Cloudflare](https://developers.cloudflare.com/privacy-gateway/); it's not possible for Divvi Up to operate the relay while providing privacy.

Read on for details about how this improves Divvi Up's privacy, how we built our solution and how you can use it in your telemetry.

## What is Oblivious HTTP?

Oblivious HTTP ([RFC 9458](https://datatracker.ietf.org/doc/rfc9458/)) is a new standard that enhances the privacy of HTTP interactions by serializing HTTP messages into a binary format and then encrypting them to a key held by the *gateway*. This is called *encapsulation*. The encapsulated messages are then sent to a *relay* server, which forwards them to the gateway. The gateway *decapsulates* the messages (decrypting them and deserializing them from binary HTTP) and finally sends them to the *target*, the ultimate recipient of the message. The gateway is generally (but not always) run by the same organization as the target, but the relay must not collude with the gateway or target except to correctly forward all encapsulated messages.

The result is that the relay gets to see the clients that participate in the protocol but not the messages they send, while the gateway and target get to see the message content but doesn't learn anything about the clients.

OHTTP is already in use in several large-scale internet applications. For example, Apple's [Private Cloud Compute](https://security.apple.com/blog/private-cloud-compute/) uses it to prevent servers from learning which clients are interacting with AI models, and Firefox uses it to enhance the privacy of their new [Review Checker](https://support.mozilla.org/en-US/kb/review-checker-review-quality).

## Composing Oblivious HTTP with DAP

Our recommendation is to use OHTTP with Divvi Up only when uploading reports to the aggregators. In this section, we'll discuss the different resources exposed by a DAP aggregator and whether it makes sense to serve them over OHTTP, to explain how we arrived at this recommendation.

In the [Distributed Aggregation Protocol](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/), individual measurements are protected using secret sharing schemes, meaning that either participating aggregator only sees shares of the sensitive data that are indistinguishable from random noise and reveal nothing about the original measurement. The collector, in turn, only receives aggregations over the contributions and thus learns neither the individual contributions nor which clients participated.

But even with DAP, both the leader and helper aggregators can learn some information about the participating clients. Potentially identifying metadata such as IP addresses, user agents and TLS ciphersuites are revealed when clients fetch HPKE configurations from either aggregator or upload reports to the leader. This could allow aggregators to fingerprint clients and associate them with DAP tasks, since the task ID appears in the request paths for uploads and (optionally) HPKE configurations.

But if we layer DAP on top of OHTTP, with the DAP aggregator acting as the OHTTP target, then the aggregators no longer learn which clients are participating at all, much less which tasks they participate in.

Our first thought might be to run all DAP interactions over OHTTP. But using OHTTP does incur some overhead in network transmission and computation. Additionally, since it requires an independently operated server, it leads to challenges like the ones we and others discussed in [The Deployment Dilemma](https://mpc.cs.berkeley.edu/blog/deployment-dilemma.html). We need to judiciously trade off privacy against cost and operational complexity when choosing where to use OHTTP.

Running either the [aggregation](https://datatracker.ietf.org/doc/html/draft-ietf-ppm-dap-11#section-4.5) or [collection](https://datatracker.ietf.org/doc/html/draft-ietf-ppm-dap-11#section-4.6) interactions over OHTTP is futile: in both cases, Divvi Up requires that the HTTP requests be authenticated with bearer tokens, which allow our servers to identify requesters regardless of any anonymizing proxies in play. You could use something like [Privacy Pass](https://datatracker.ietf.org/wg/privacypass/) to anonymously authenticate (and see below for more discussion of this in the client setting), but the objective of DAP is to protect the privacy of clients. We're less concerned with hiding which collectors and aggregators are participating in a task, since those actors have deliberately entered into a relationship with Divvi Up anyway.

What's more nuanced is DAP HPKE configuration fetching. Current drafts of DAP allow adding an optional `task_id` query parameter to HPKE configuration requests, so that aggregators can use distinct HPKE configuration for different tasks. But this means that clients reveal interest in a task when they fetch HPKE configurations. DAP also allows aggregators to use the same set of HPKE configurations across all tasks, which removes that particular information leak. But aggregators would still get to see which clients may participate in some DAP task when they fetch keys, which might nudge us toward serving HPKE configurations over OHTTP.

But let's take a step back and consider how OHTTP itself works: before a client can encapsulate messages to the OHTTP gateway, it must obtain an OHTTP key configuration (used to encrypt messages so that the OHTTP relay can't see them; not to be confused with the DAP HPKE configuration used to conceal input shares from the leader). In Divvi Up's deployment, we serve OHTTP keys from the gateway itself. This way, clients can fetch the keys over HTTPS from a subdomain of a trusted name like `api.divviup.org` and have confidence that only Divvi Up will be able to decapsulate OHTTP messages.

The upshot is that even if we served DAP HPKE configurations over OHTTP, the server would still see the exact same clients' metadata when they fetch OHTTP key configurations. So no new information is learned by observing DAP HPKE configuration fetches, and not using OHTTP there enables optimizations like serving them over a content distribution network. Further, a client fetching keys for OHTTP and then DAP reveals that they *might* participate in a DAP task in the future, but it doesn't guarantee it, and doesn't reveal which task.

## Divvi Up's deployment

We believe in using open standards and open source as a means of being transparent and earning trust, but another benefit is that it means we can often deploy open source implementations to rapidly achieve goals instead of having to write something ourselves.

In the case of our OHTTP gateway, we use [`privacy-server-gateway-go`](https://github.com/cloudflare/privacy-gateway-server-go), maintained by [Cloudflare Research](https://research.cloudflare.com/). As the name indicates, it's written in Go, and thus is memory safe, in keeping with [our organization's values](https://memorysafety.org). While adopting it, we contributed [a number of fixes and improvements](https://github.com/cloudflare/privacy-gateway-server-go/pulls?q=is%3Apr+is%3Aclosed+author%3Atgeoghegan) to `privacy-server-gateway-go`, adding features like Prometheus metrics and configurable support for rewriting target URIs.

Divvi Up runs on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), and we use our own [Janus](https://github.com/divviup/janus) to run either a DAP leader or helper. Each aggregator replica runs in its own Kubernetes pod, so the sidecar container pattern is a natural way to run the gateway. In fact, we already run a rate limiting reverse proxy as a sidecar in our Internet facing deployments.

The advantage of the sidecar container pattern is that within a pod, all containers share the same network namespace, meaning that network communication over the loopback interface is cheap. We just configure `privacy-server-gateway-go` to forward decapsulated traffic to another port on `localhost` where the reverse proxy is listening and everything just works -- none of the components behind the OHTTP gateway need to be aware that they're involved in OHTTP.

This diagram illustrates how we use a Kubernetes ingress to route traffic to one of two Kubernetes services based on the request path: `/gateway` or `/ohttp-keys` goes to the OHTTP gateway, while other traffic (e.g., DAP uploads or aggregation jobs) go to the regular reverse proxy and are handled or rejected there.

![](/images/blog/Blog-2024-07-25-Graphic.png)

## How to use it

The Oblivious HTTP gateway is already available in Divvi Up's production environment. OHTTP key configurations conforming to [RFC 9458 section 3](https://datatracker.ietf.org/doc/html/rfc9458#section-3) can be fetched from <https://dap-09-3.api.divviup.org/ohttp-keys>, and the gateway which can decapsulate messages encrypted to those keys is reachable at <https://dap-09-3.api.divviup.org/gateway>.

[`janus_client`](https://docs.rs/janus_client/0.7.24/janus_client/index.html), Divvi Up's official Rust client, supports this via the [`ClientBuilder::with_ohttp_config method`](https://docs.rs/janus_client/0.7.24/janus_client/struct.ClientBuilder.html#method.with_ohttp_config).

```
#[tokio::main]
async  fn  main()  {
    // Create a task via the Divvi Up console, app or CLI and provide
    // its ID here.
    let task_id = random();
    let client = janus_client::Client::builder(
        task_id,
        Url::parse("https://dap-09-3.api.divviup.org/").unwrap(),
        Url::parse("https://helper.example.com/").unwrap(),
        Duration::from_seconds(1),
        Prio3Count::new_count(2).unwrap(),
    )
    .with_ohttp_config(janus_client::OhttpConfig  {
        key_configs: Url::parse("https://dap-09-3.api.divviup.org/ohttp-keys").unwrap(),
        // Work with an OHTTP relay vendor and provide the relay URI
        // here
        relay: Url::parse("https://ohttp-relay.example.com").unwrap(),
    })
    .build()
    .await
    .unwrap();
    
    // Uploads will be encapsulated to the key configurations and
    // proxied via the relay
    client.upload(&true).await.unwrap();
}
```

If you don't yet have an OHTTP relay but want to experiment with this feature, you can simply provide the gateway URI as the relay when constructing `janus_client::OhttpConfig`. The client will still encapsulate messages and the gateway will decapsulate and forward them as usual, but of course this provides no privacy guarantees because the clients interact directly with the gateway.

If you are working in a different language or not using the Divvi Up client, then you can use any number of open source Oblivious HTTP implementations to construct suitable requests:

-   The [ohttp Rust crate](https://github.com/martinthomson/ohttp) is what `janus_client` uses
-   [ohttp-go](https://github.com/chris-wood/ohttp-go) is a Go implementation
-   [ohttp-js](https://github.com/chris-wood/ohttp-js?tab=readme-ov-file) for JavaScript or TypeScript projects

## What's next: Privacy Pass and differential privacy

On its own, the Distributed Aggregation Protocol provides strong privacy guarantees to users participating in telemetry. But as we've seen in this post, we can significantly improve the privacy posture by layering DAP on top of OHTTP. What's remarkable is that neither protocol needed to be explicitly designed in terms of the other in order to be composable in this manner, because both are built around HTTP. And the unsung hero of all this is [Transport Layer Security](https://datatracker.ietf.org/doc/html/rfc8446), which was providing the privacy-critical properties of confidentiality, integrity and authenticity to higher level protocols like HTTP for decades before we even started designing DAP.

Going further, we can imagine layering on other technologies to make Divvi Up even more private. Currently, Divvi Up doesn't have any way for DAP clients to authenticate to aggregators. As we discussed earlier, many authentication schemes reliably identify the client and so undermine all the other privacy properties we want to have. But we can use [Privacy Pass](https://datatracker.ietf.org/wg/privacypass/) to verify that clients are legitimate (e.g., an authentic hardware device manufactured by the task deployer uploading real telemetry and not just a script uploading synthetic data) without revealing anything else about the client. We anticipate that integrating Privacy Pass with DAP should be a simple matter of adding an extension to reports containing the token.

Beyond that, we are working with our colleagues in the research and engineering communities on augmenting the Distributed Aggregation Protocol with differential privacy (DP). Explaining DP is beyond the scope of this post, but the idea is that we can add carefully tuned amounts of random noise to aggregations to protect the privacy of individual contributions against attacks involving comparing multiple aggregations over nearly identical populations of clients. Tuning DP requires much more intimate knowledge of the nature of the data, which means it doesn't compose with DAP as modularly as OHTTP or Privacy Pass, but the resulting protections are more than worth the investment of research and development. Stay tuned for future posts about how Divvi Up and the [Privacy Preserving Measurement working group](https://datatracker.ietf.org/wg/ppm/about/) are thinking about differential privacy.

*Want to learn more about Divvi Up and privacy respecting statistics collection? Get started with Divvi Up, in under 5 minutes, with Docker and your command line. Our new tutorial will have you divvi'ing user statistics in no time. <https://docs.divviup.org/command-line-tutorial>*