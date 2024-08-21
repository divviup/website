---
author: Caro Hadad, Horizontal
date: 2024-08-21T00:00:00Z
slug: horizontal-tella
title: "Privacy-preserving Telemetry for Android Apps"
excerpt: "A human rights defender nonprofit adds Divvi Up to their Android App."
---




<div class="card border-0">
    <div class="pt-4 pb-4 blockquote-intro-offset">
        <blockquote class="blockquote intro">
            <span class="quote"></span>
            <div class="intro-text">
                <p class="font-italic">Divvi Up helps organizations improve their products while fully respecting user privacy. We're excited to share this post from our partner <a href="https://wearehorizontal.org/index">Horizontal,</a> who has integrated Divvi Up into their <a href="https://tella-app.org/">Tella product</a> with their release of <a href="https://tella-app.org/releases/#android-tella-2100-185---released-on-aug-5-2024">Tella Android 2.10.0</a>. As an <a href="https://divviup.org/blog/horizontal/">early adopter</a> of our privacy-preserving telemetry service in their Shira platform, Horizontal continues to lead the way in protecting user data while gaining valuable insights.</p>
                <p class="font-italic lh-170">In challenging environments with limited Internet connectivity, or in the face of repression, Tella makes it easier and safer to document human rights violations and collect supporting data. We're proud to support Horizontal's critical mission and look forward to continuing our partnership. We'd like to thank Open Technology Fund for introducing us to Horizontal!</p>
                <div class="blockquote-footer mt-3"><cite title="Source Title">Brandon Pitman, Divvi Up Engineer</cite></div>
            </div>
        </blockquote>
    </div>
</div>

<img src="/images/blog/2024-08-22-tella-app.svg" alt="Tella App" class="tella-blog-post-image">

Introducing Privacy-Preserving Analytics (opt-in)
-------------------------------------------------

One of the features included in this release is our new privacy-preserving analytics system. Users now can choose to share anonymous usage data to help us improve Tella. We understand how important privacy is to our users, so we chose [Divvi Up](https://divviup.org/), a privacy-respecting telemetry service by the [Internet Security Research Group (ISRG)](https://www.abetterinternet.org/), the nonprofit organization behind [Let's Encrypt](https://letsencrypt.org/). We [were the first implementer](https://divviup.org/blog/horizontal/) of Divvi Up in [Shira](https://shira.app/) (another of our products) and we are very happy with how it is working, so now we added it also to Tella.

Here are some key details about the implementation:

-   **All data is anonymous and aggregated**: the Divvi Up library splits the data into two anonymized and encrypted shares and uploads each share to different data share processors (one hosted by ISRG and one hosted by us) that do not share data with each other. This way, only partial information about the original data is revealed to either processor.

-   **Even if we wanted to, we wouldn't be able to get the whole data**: It's not possible to construct the whole data with only one share. Each processor aggregates its data shares into a partial sum. The partial sums can then be combined into a final aggregation, permitting useful statistics over the whole body of data while revealing minimal information about individual participants. Extensive technical documentation about how Divvi Up works can be found [here](https://docs.divviup.org/).

-   **We collect as little data as possible**: Even though all data is anonymized, we always minimize the amount of data we collect. All metrics collected are listed [here](https://tella-app.org/security-and-privacy/#analytics).

[View the original post on Horizontal's blog](https://blog.wearehorizontal.org/privacy-preserving-analytics-added-to-tella-android-2/)

*We'd like to thank Open Technology Fund, Internet Society Foundation, Omidyar Network, Ford Foundation, The Robert Wood Johnson Foundation, Google, and Meta for supporting the development of the novel privacy-respecting technology behind Divvi Up.*

*If you have questions about using Divvi Up for your telemetry use case, please get in touch at <contact@divviup.org>. If you want to learn more about Divvi Up and privacy respecting statistics collection, get started with Divvi Up in under 5 minutes with Docker and your command line. Our new tutorial will have you divvi'ing user statistics in no time. <https://docs.divviup.org/command-line-tutorial>*