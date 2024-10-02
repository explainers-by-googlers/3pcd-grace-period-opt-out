# **Explainer for the Third-party Cookie Grace Period Opt-Out**

This explainer outlines a mechanism to allow sites to opt out of Chrome's [third-party cookie grace period](https://developers.google.com/privacy-sandbox/cookies/temporary-exceptions/grace-period) for a percentage of users. Feedback is welcome. 


## Authors

Anton Maliev \
Jonathan Njeunje


## Introduction

While browsers are actively working to remove support for third-party cookies, many web providers have also implemented various mitigations for the smoothest transition and reduction of user-facing (non-ads) breakages. Sites have the ability to [report new breakages](https://developers.google.com/privacy-sandbox/blog/3pc-exceptions-update?hl=en#details_of_the_updated_grace_period) which upon successful verification are granted a grace period that allows them continuous access to third-party cookies.

Although developers can use a [testing setup](https://developers.google.com/privacy-sandbox/3pcd/prepare/test-for-breakage) to locally verify their site works with third-party cookies disabled, there is currently no mechanism to safely do the same in production, other than contacting Chrome and requesting to be taken off the grace period entirely. This is a suboptimal process as any bugs that are not caught in local testing will be instantly launched to 100% of users, and developers would need to reach back to Chrome to be re-added to the grace period. This is prone to increased user-facing outages, and doesn’t scale well. Overall, difficulties with production testing is a barrier to sites migrating off the grace period.

This explainer details a self-service system that gives sites the ability to opt-out of the grace period for a certain percentage of clients. Once the developers fully have their site working without third-party cookies, they can use this mechanism to test these changes with a sample of users in production. This will accelerate the overall migrating process by helping sites confidently move off the grace period and onto better solutions. This explainer examines use cases that will benefit from this system, as well as its privacy and security implications.


## Goals



* Design a mechanism for sites to confidently and swiftly migrate to [long-term privacy-preserving APIs](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared), by gradually opting out of the grace period in a self service fashion with minimal involvement from Chrome.
* Provide a revert mechanism for sites on their grace period opt-out phase to quickly and smoothly re-enable their grace period as needed.
* Encourage sites to confidently remove their dependence on the grace period as quickly as they are able to.


## Non-goals



* This proposal does not require sites to implement the .well-known file in order to take advantage of the grace period, which is currently in effect.
* This proposal is not intended to be a permanent feature of the web. It only applies to sites on the grace period, so it will not be required once Chrome stops providing the grace period.
* This proposal does not specify deadlines, globally or per-site, for turning down any short/long term mitigation, and removing third-party cookies entirely. We intend for the grace period to be a temporary mitigation, but we will need continued alignment with both Chrome and site developers to establish timelines, which are not in scope for this explainer.
* This proposal does not introduce new web APIs for migrating away from third-party cookies in the long term. There are [other APIs](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared) available for this purpose, e.g. [FedCM](https://fedidcg.github.io/FedCM/), [Storage Access API](https://privacycg.github.io/storage-access/), and [Fenced Frames](https://wicg.github.io/fenced-frame/).


## Browser Behavior Proposal

Once a site is approved for the grace period, it can add a resource to its [.well-known directory](https://datatracker.ietf.org/doc/html/rfc8615) to indicate the desired opt-out percentage. The URI is registered in the [well-known URI directory](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) as .well-known/tpcd/grace-period.json, and its spec is defined [in this repository](https://github.com/explainers-by-googlers/3pcd-deprecation-trial-staged-rollout/blob/main/well-known-specification.md). The file is hosted under the same domain as listed in the deprecation trial application. A sample file would look like this:

```
{
  "ThirdPartyOptOutPercentage": 50
}
```

This example represents a 50% Opt-Out from the grace period for the third-party embedded context.

The browser will implement server infrastructure to fetch updated opt-out percentage values for all sites in the trial with a 24-48 hour turnaround. The percentage value will determine what percentage of browser clients are randomly selected to have the grace period disabled for a particular site. These cohort selections will persist for a client, up until they reinstall the browser or clear site storage, or the site updates its percentage value again.

Site developers will also have a way of inspecting the rollout of their opt-out percentage value config, dependent on browser implementation. In third-party use cases where both the top-level and embedded sites are rolling out a privacy-preserving alternative, the maximum of their percentage values is used.


## Key Use Cases

The following flows detail possible development journeys for sites incorporating a replacement for the grace period.


### Use Case 1: Top-level Site Grace Period Grant



1. Site developer ExampleTop detects breakage due to third-party cookie turndown on their site www.example.com.
2. ExampleTop follows the [documentation](https://developers.google.com/privacy-sandbox/blog/3pc-exceptions-update?hl=en#details_of_the_updated_grace_period) for reporting for the breakage. They apply for the site pattern [*.]example.com to grant all subdomains the ability to have third-party cookies access.
3. After internal review by Chrome, their application is approved. [*.]example.com is added to the grace period and the user-facing breakage is mitigated within 24-48 hours with default fallback opt-outs of 0.
4. ExampleTop integrates their short/long term solution of choice to all affected cookies on [*.]example.com, and performs local testing.
5. ExampleTop creates the .well-known resource with an initial FirstPartyOptOutPercentage value of 10, and hosts it on example.com/.well-known/tpcd/grace-period.json.
6. ExampleTop gradually increments FirstPartyOptOutPercentage towards 100 while monitoring breakage reports, and at 100 the grace period is fully disabled.
7. Chrome will pick up every change to the .well-known resource and effectively turn down the grace period on a proportionate percentage of clients.
8. At this point, ExampleTop should maintain serving the grace-period.json. And use it to take further (reverting) action as needed.
9. In the long run, ExampleTop completely migrates to one of the [long-term privacy-preserving APIs](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared) offered by the browser.
10. ExampleTop can stop serving the grace-period.json once any of the following are true:
    1. The grace period grant reached expiry.
    2. The grace period grant was revoked with the entry deleted from Chrome servers.


### Use Case 2: Embedded Site Grace Period Grant



1. Site developer ExampleEmbed detects breakage due to third-party cookie turndown on their third-party plugin embed.example.com.
2. ExampleEmbed follows the [documentation](https://developers.google.com/privacy-sandbox/blog/3pc-exceptions-update?hl=en#details_of_the_updated_grace_period) for reporting for the breakage. They apply for the site pattern embed.example.com to grant only the exact subdomain embed.example.com the ability to regain third-party cookies access.
3. After internal review by Chrome, their application is approved. embed.example.com is added to the grace period and the user-facing breakage is mitigated within 24-48 hours with default fallback opt-outs of 0.
4. ExampleEmbed creates the .well-known resource with an initial ThirdPartyOptOutPercentage value of 0, and hosts it on embed.example.com/.well-known/tpcd/grace-period.json.
5. ExampleEmbed migrates their third-party cookies to instead use [Partitioned Storage](https://developers.google.com/privacy-sandbox/3pcd/chips), a long-term privacy-preserving API, and performs local testing.
6. ExampleEmbed increments ThirdPartyOptOutPercentage to 10. However, they notice an increase in user-facing breakage and identify a bug in their CHIPS adoption. They revert ThirdPartyOptOutPercentage to 0 while the fix is rolled out. The breakage is mitigated within 24-48 hours of the resource update.
7. Once the fix is released and more local validation is added, ExampleEmbed restarts the ramp up of ThirdPartyOptOutPercentage towards 100.
8. At this point, ExampleEmbed should maintain serving the grace-period.json. And use it to take further (reverting) action as needed.
9. ExampleEmbed can stop serving the grace-period.json once any of the following are true:
    1. The grace period grant reached expiry.
    2. The grace period grant was revoked with the entry deleted from Chrome servers.


## Considered Alternatives

Possible alternatives to the proposal above include:



1. Extend the current grace period with a “opt-out percentage” flag in the internal browser configuration. This adds support for a staged opt-out, but it’s not self-service and would require close and ongoing alignment between the developer and Chrome, which doesn’t scale well. This alignment also increases the SLA duration for rolling out updates, including rollbacks in case of breakage.
2. Define a .well-known file as specified above, but have each client fetch the resource, rather than Chrome. This decreases the latency for opt-out percentage value updates, and potentially gives the site more control over its opt-out populations. However, it has significant downsides:
* It would expose client browsing history via its network requests to specific .well-known resources.
* It would require requests to the domain of embedded sites (if there is a third-party grace period active) which adds new cross-site information leakage through timing attacks, etc.
* It would greatly increase the traffic load to the .well-known resource and could overload its server.
* It would add a performance cost due to an additional request for each client navigation, which could slow down the browser.


## Privacy and Security Considerations

Although there aren't inherent security risks with fetching .well-known resources from server-side infrastructure, there is a potential vulnerability of fetching bad data which could be used to exploit Google’s servers. We will mitigate this by rate-limiting HTTP requests, by scoping the requests to only the specific .well-known file, and by not storing any irrelevant or invalid information from the file - only what matches the [spec](https://github.com/explainers-by-googlers/3pcd-deprecation-trial-staged-rollout/blob/main/well-known-specification.md).

Assigning every user to a randomized cohort for each site in the grace period creates a stable identifier with a high level of entropy for each client, which could be a privacy concern if it is used to identify distinct users. We are mitigating this risk factor by ensuring these cohorts are cleared on site data deletion. These cohorts are also independent and only apply to one site, so it’s not possible for a site to join the cohorts and construct a cross-site identifier for tracking.

We are also defining an internal override flag for each site on the grace period. This would be set only in cases where the site developer is unable to edit the site’s .well-known resource, or if there is a broader issue with the system that fetches and updates the resources. Access to this override flag will be strictly gated using a two-party ACL system.


## Stakeholder Feedback / Opposition

TBD - This proposal is pending feedback from the appropriate WGs.


## References & acknowledgements

Thank you for design input, feedback, and review from: \
Ben Kelly \
Artur Janc \
Christian Dullweber
