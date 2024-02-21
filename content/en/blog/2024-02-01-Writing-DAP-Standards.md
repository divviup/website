---
author: Tim Geoghegan
date: 2024-02-20T00:00:00Z
actual-date: 2024-02-21T00:00:00Z
slug: Writing-DAP-Standards
title: "The Distributed Aggregation Protocol: A First-Time Editor's View of Writing Standards at the IETF"
excerpt: "What we’ve learned and how we’ve evolved Divvi Up through the process of writing an Internet Standard in the IETF."
---

[Divvi Up](https://divviup.org/) is a privacy-preserving system for aggregating metrics and telemetry. It's powered by novel cryptography that protects the privacy of individuals contributing measurements to aggregations by splitting the data across two non-colluding servers. Because we want this system to work across multiple servers operated by multiple teams, possibly using distinct implementations, we need a robust protocol specification.

Divvi Up engineers have been involved in writing and iterating on the [Distributed Aggregation Protocol](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/) (DAP), along with many collaborators from industry and academia, since early 2021. While our colleagues at [Let's Encrypt](https://letsencrypt.org) have extensive experience with standards work, DAP is the first time the Divvi Up team has been involved with the [Internet Engineering Task Force](https://ietf.org) (IETF).

Now, three years and nine revisions of draft-ietf-ppm-dap later, the DAP editors are hopeful that we're in the home stretch of protocol development. In this post, we're going to look at some key milestones in the development of the document and then discuss what we believe remains to be done. Along the way, we'll try to demystify some IETF procedural points, in hopes of making participation in the Internet standards process seem less daunting to you, dear reader.

Working group formation and draft adoption
------------------------------------------

Documents at the IETF are owned by working groups. The primary venue for group discussion is a good old fashioned email list managed by the IETF, as well as sessions at the plenary meetings, held in person and online three times a year. While there's a registration fee for attending a plenary, anyone can read or post to the working group lists for free, and that's the only requirement for joining. Working groups will also usually have two chairs, who are responsible for moderating discussions, scheduling and facilitating sessions during plenaries, shepherding documents through the IETF process, and resolving conflicts as needed (we'll talk about "rough consensus" later).

If the new document doesn't fit in an existing working group's charter, then the first step to writing a standard is to form a new group. To do that, interested parties will request that a special session called a [Birds of a Feather](https://www.ietf.org/how/bofs/) (BOF) be held during a plenary meeting. A BOF session allows the IETF to gauge interest in a new technology area, to see if anyone is interested in implementing and deploying it and in particular whether enough people are willing to help write standards documents.

BOFs also typically discuss what the working group's charter will be. A charter is a crisp statement of the group's goals, like documents to be delivered. But it can also declare some problems as outside the working group's scope. This is an important tool that working group chairs use to keep the group focused on solvable problems and to prevent feature creep.

The BOF for Privacy Respecting Incorporation of Values (PRIV; [clever](https://datatracker.ietf.org/wg/ohai/about/)  [acronyms](https://datatracker.ietf.org/wg/mimi/documents/)  [are](https://datatracker.ietf.org/wg/asap/documents/)  [an](https://datatracker.ietf.org/wg/masque/documents/)  [unofficial](https://datatracker.ietf.org/wg/acme/documents/)  [IETF](https://datatracker.ietf.org/wg/quic/about/)  [tradition](https://datatracker.ietf.org/wg/cats/documents/)), which would later become Privacy Preserving Metrics (PPM), was held during [IETF 112 in November 2021](https://datatracker.ietf.org/meeting/112/agenda). At the time, travel restrictions due to the COVID-19 pandemic were in effect, so that meeting was held completely online. Today, the IETF is back to holding meetings in person, but remote attendees are still welcome and can mostly participate just like everyone else.

Once a working group is formed, the next step is to get a document adopted by it. Document authors will informally gather and collaborate on an initial draft of a standards document. These days, it's common for the document to be hosted on GitHub (see for instance the [GitHub repository for DAP](https://github.com/ietf-wg-ppm/draft-ietf-ppm-dap)), but the [IETF Datatracker](https://datatracker.ietf.org/) is the authoritative home of drafts and published documents. Adoption of a draft doesn't mean that the document is finished or meets IETF standards. It simply means that the working group agrees that it is addressing the right problems and that it's on the right track. The draft will be iterated upon, probably many times, before becoming a proposed standard.

Engineers from Divvi Up, Cloudflare Research and Mozilla began work on what would become DAP in February of 2021. We called the earliest versions Private Data Aggregation and then Privacy Preserving Measurement (PPM). But by IETF 113, in March of 2022, the PPM name had been taken by the newly-minted working group, and so the chairs renamed the protocol to Distributed Aggregation Protocol and draft-ietf-ppm-dap-00 was adopted.

{{< svg "static/images/blog/2024-02-24-timeline.svg" >}}

Preview
-------

![](/images/blog/2024-02-24-timeline.png)

Iterating on drafts
-------------------

Pre-adoption, the document's authors can write whatever they want. However, once the document is owned by the working group, changes to it should reflect the consensus of the group's members. Groups will name a team of generally no more than five people to edit the document. While those editors have significant influence on the document by virtue of doing most of the writing, they can't unilaterally make changes. There's two obvious ways to ascertain whether a working group has consensus on a question: you could take a vote, or you could require unanimous agreement. The former isn't ideal because a simple majority can mean that up to half of the working group's input has been ignored. Unanimity, on the other hand, is rarely practical because it would allow intransigent individuals to block progress forever. Broadly, there's a tradeoff here between incorporating everyone's contributions and pragmatism.

The IETF resolves this tension using a notion it calls "rough consensus". [RFC 7282](https://datatracker.ietf.org/doc/html/rfc7282) discusses this in detail, but the idea is that decisions should reflect the overall sense of the group. That's a higher bar than the simple majority of a vote, but also allows for some small minority to dissent, so long as their objections have been heard and considered by the group. Deciding if the threshold of rough consensus has been met is left up to the working group chairs. This is quite informal and by no means perfect, but surprisingly effective and has served the IETF well for several decades now.

In DAP, we've successfully used this process to evaluate several major changes to the protocol text. For example, draft 4 drastically remixed the API exposed by DAP aggregators to make better use of HTTP primitives as laid out in [RFC 9110](https://datatracker.ietf.org/doc/rfc9110/) and [RFC 9205](https://datatracker.ietf.org/doc/rfc9205/), but we used rough consensus so that we could avoid getting stuck on protocol design minutiae that wouldn't ultimately make a difference. More recently, we requested a review from the IETF's [HTTP directorate](https://datatracker.ietf.org/group/httpdir/reviews/). We got loads of excellent suggestions from Mark Nottingham, the httpdir reviewer, but didn't wind up implementing all of them. Rough consensus was a great way for the PPM working group to decide which suggestions to take. Those changes will appear in draft 10, which should be published in time for IETF 119 Brisbane in March of this year.

Interop testing
---------------

The other half of the IETF's mantra is "running code". The wonderful thing about software engineering relative to its sibling disciplines is how easy it is to try out new ideas: writing a test program is much cheaper than building a test bridge. And the best way to prove that a protocol is sound or doesn't contradict itself is to go implement it.

From the very earliest days of PPM and later DAP, the Divvi Up team has maintained working implementations of the protocol. At first, we had [ppm-prototype](https://github.com/tgeoghegan/ppm-prototype), which was quickly obsoleted by [Janus](https://github.com/divviup/janus), Divvi Up's production-quality implementation of the DAP leader and helper roles. Janus also provides libraries and utilities that implement the client and collector roles. Besides helping us prove out the soundness of DAP, Janus also powers Divvi Up's live deployments, such as [our ongoing experimental collaboration with Mozilla Firefox](https://divviup.org/blog/divvi-up-in-firefox/). You can check out the rest of our open source projects [here](https://divviup.org/about/#open-source).

But the true purpose of a specification is to allow multiple independent implementations to interoperate. For example, our distinguished colleagues over at Cloudflare Research have built their own DAP implementation called [Daphne](https://github.com/cloudflare/daphne). Daphne has some significant architectural differences from Janus -- for instance, it doesn't use a monolithic SQL database but rather Cloudflare's [Durable Objects](https://developers.cloudflare.com/durable-objects/) -- and it's extremely valuable because it proves that two independent teams working from the same specification can arrive at distinct implementations that can work together.

While we've run a variety of integration tests between Daphne and Janus over the years, we found that we wanted a more lightweight mechanism for testing the two implementations against each other. To that end, Divvi Up engineer David Cook wrote another protocol document in the PPM working group describing the [DAP Interoperation Test Design](https://datatracker.ietf.org/doc/draft-dcook-ppm-dap-interop-test-design/). This protocol specifies a test orchestration API that DAP implementations can provide, which allows a test harness -- like the one in the [janus_integration_tests package](https://github.com/divviup/janus/tree/main/integration_tests) -- to efficiently and affordably test any two conforming implementations against each other. This means that we've been able to run such tests automatically as part of the standard Janus CI suite, verifying compatibility with other implementations every time we merge a change.

This work was inspired by the [quic-interop-runner](https://github.com/quic-interop/quic-interop-runner), which was used by the QUIC working group to validate a diverse set of implementations of [RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000).

Proposed standard
-----------------

In the two years since the PPM working group was formed, DAP has made some great strides toward becoming a standard. But between some open questions about the protocol, as well as the process that any IETF standards document has to go through, we still have a ways to go.

DAP composes some fairly novel concepts in cryptography and privacy, like multi-party computation (MPC) and differential privacy. These technologies are just recently being widely deployed on the Internet, and to our knowledge, neither has ever been standardized in a body like the IETF before. So on top of the usual, considerable difficulties of writing Internet standards, we are also experimenting with the scalability of MPC. In particular, DAP has some scalability challenges around gathering individual measurements into batches over which aggregations are computed. We haven't nailed down how to efficiently provide robust anti-replay protections and how to make sure that collectors can't construct batches in a way that could isolate and deanonymize individual contributions.

As we run DAP on significant volumes of data for the first time, Divvi Up engineers are learning about where the performance bottlenecks are and working to improve the efficiency of our implementation. Some of our findings about how to run DAP efficiently may be reflected in upcoming drafts of the protocol as requirements or guidance for other DAP implementations and deployments.

Another area of DAP that may evolve as we gain more practical experience is the handling of [Verifiable Distributed Aggregation Functions](https://datatracker.ietf.org/doc/draft-irtf-cfrg-vdaf/) (VDAFs) besides Prio. VDAFs are a cryptographic abstraction that lets DAP coordinate the execution of multiple different aggregation primitives. The Prio3 family of VDAFs excels at aggregating over numerical measurements of known size, but you only ever need to aggregate a given batch of reports once. Other VDAFs may allow making multiple queries against a batch. For example, [Mastic](https://datatracker.ietf.org/doc/draft-mouris-cfrg-mastic/00/) is designed for finding out how many values among a large set of strings begin with some prefix. You might be aggregating over the URLs that users of a web browser visit, and you want to know how many of them start with "divviup.org". But once you have learned that, you might want to refine your query to find out how many start with "divviup.org/foo", and then "divviup.org/bar". Mastic allows you to run multiple aggregations over the same batch but with different string prefixes.

Currently, DAP would require that the entire batch of report shares be retransmitted from leader to helper every time the batch is aggregated, even if the shares in question were already sent earlier. We're considering extending DAP so that report shares can be transmitted to the helper independently from aggregation jobs, which should significantly reduce the overall bandwidth utilization when running VDAFs like Mastic. However, we need more practical experience with that VDAF to help us decide if the performance improvement is worth the extra protocol complexity. We're hoping to run some relevant experiments soon.

At some point, the working group will decide that DAP is finished. Naturally it won't be perfect: one of the challenges of any technology project is accepting that it won't be the last word in the problem domain. Just as it took the Internet community a few tries to arrive at version 1.3 of the Transport Layer Security standard, the PPM working group probably has quite a few years of trying and learning and trying again ahead of us. In any case, DAP will have several more steps of IETF review and approval to go through before it can be ratified as an IETF proposed standard, which is the somewhat ironic "final" status conferred on documents after they've undergone years of development, iteration and review.

Conclusion
----------

In this post, we've discussed some of the history of the Distributed Aggregation Protocol and the Privacy Preserving Measurement working group that is producing it. We also explored how working groups are formed and how standards documents come together at the IETF. There's lots of programmers, engineers, technologists, advocates and other Internet users out there who have a lot to contribute to the Internet standards process, but understandably may be intimidated by what can seem like an impenetrable institution with lots of arcane procedures and inside-baseball jargon and jokes. And it's true, the IETF has accumulated a lot of that. But the IETF can't function at all unless new participants come and join working groups, and it can't be effective at its mission of "making the Internet work better" unless its participants reflect the diversity of the Internet's users.

As we noted at the start of this post, the Divvi Up engineering team had no prior experience of standards work before writing DAP, but we benefited from the advice and guidance of several well-established IETF participants. Now, we want to pay that forward by welcoming a further cohort of new IETF participants. We hope that this discussion will make the standards process seem more approachable. Come say hi [on the mailing list](mailto:ppm@ietf.org) or at the next IETF plenary meeting and we'll welcome you to the IETF community!

If you're interested in becoming a Divvi Up user, please join our [waitlist](https://divviup.org/get-involved#waitlist).

<script>
document.addEventListener('DOMContentLoaded', function() {
    var clickableAreas = document.querySelectorAll('svg a[id^="link-"]');
    clickableAreas.forEach(function(element) {
        var bbox = element.getBBox();
        var rect = document.createElementNS("http://www.w3.org/2000/svg", "rect");
        rect.setAttribute("x", bbox.x);
        rect.setAttribute("y", bbox.y);
        rect.setAttribute("width", bbox.width);
        rect.setAttribute("height", bbox.height);
        rect.setAttribute("fill", "red");
        rect.setAttribute("fill-opacity", "0.5"); // 50% opacity for debugging visibility
        element.insertBefore(rect, element.firstChild);

        element.addEventListener('click', function(event) {
            event.preventDefault();
            var url = this.getAttribute('href');
            window.open(url, '_blank')
        });
    });
});
</script>