# Explainer for the Third-party Cookie Grace Period Opt-Out

This proposal is an early design sketch by the Chrome Privacy Sandbox team to describe the problem below and solicit feedback on the proposed solution. It has not been approved to ship in Chrome.

## Authors
Anton Maliev\
Jonathan Njeunje

## Introduction
While browsers are actively working to remove support for third-party cookies, many have also implemented various mitigations to reduce user-facing breakage and allow sites to have a smooth transition. Google Chrome is managing two deprecation trials - [one for top-level sites](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/first-party-deprecation-trial) and [one for embedded sites](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/third-party-deprecation-trial) - which site developers can apply for. Sites in these trials receive a token which they need to deploy to temporarily exempt them from the third-party cookie restrictions. Once a site is approved for the trial, they are also added to a short-term grace period which is intended to mitigate breakage until the token is launched.

Although developers can use a [testing setup](https://developers.google.com/privacy-sandbox/3pcd/prepare/test-for-breakage) to locally verify their token launch, there is currently no mechanism for sites to turn off the grace period in production, other than contacting Chrome and requesting to be taken off the grace period entirely. This is a suboptimal process as any bugs that are not caught in local testing will be launched to 100% of users, and developers would need to contact Chrome again to be re-added to the grace period. This is prone to increased user-facing outages, and doesn’t scale well.   Overall, difficulty with production testing is a barrier to sites migrating off the grace period.

This explainer details a self-service system that gives sites the ability to opt-out of the grace period for a certain percentage of clients. Once the site fully rolls out its deprecation trial tokens, or a privacy-preserving cookie alternative, it can use this mechanism to test this change with a sample of users in production. This will accelerate the overall deprecation process by responsibly helping sites move off the grace period and onto better solutions. This explainer examines use cases that will benefit from this system, as well as its privacy and security implications.

## Goals
- Design a mechanism for sites on the deprecation trial to conduct a staged enforcement of their token, or a [long-term privacy-preserving API](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared), by gradually disabling the grace period, with minimal involvement from Chrome.
- Design a mechanism for sites on the grace period opt-out and re-enable the grace period as quickly and smoothly as possible.
- Encourage sites to remove their dependence on the grace period as quickly as they are able to.

## Non-goals
- This proposal does not require sites to implement the .well-known file in order to take advantage of the grace period, which is currently in effect.
- This proposal is not intended to be a permanent feature of the web. It only applies to sites on the grace period, so it will not be required once Chrome stops providing the grace period.
- This proposal does not specify deadlines, globally or per-site, for turning down the deprecation trial and removing third-party cookies entirely. We intend for the grace period to be a temporary mitigation, but we will need continued alignment with both Chrome and site developers to establish timelines, which are not in scope for this explainer.
- This proposal does not introduce new web APIs for migrating away from third-party cookies in the long term. There are [other APIs](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared) available for this purpose, e.g. [FedCM](https://fedidcg.github.io/FedCM/), [Storage Access API](https://privacycg.github.io/storage-access/), and [Fenced Frames](https://wicg.github.io/fenced-frame/).

## Browser Behavior Proposal
Once a site is approved for either deprecation trial, it can add a resource to its [.well-known directory](https://datatracker.ietf.org/doc/html/rfc8615) to indicate the desired opt-out percentage. The URI is registered in the [well-known URI directory](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) as .well-known/tpcd/grace-period.json, and its spec is defined [in this repository](https://github.com/explainers-by-googlers/3pcd-deprecation-trial-staged-rollout/blob/main/well-known-specification.md). The file is hosted under the same domain as listed in the deprecation trial application. A sample file would look like this:

```
{
  "ThirdPartyOptOutPercentage": 50 
  // Represents a 50% Opt-Out from the grace period for the third-party (embedded) deprecation trial.
}
```

The browser will implement server infrastructure to fetch updated opt-out percentage values for all sites in the trial with a 24-48 hour turnaround. The percentage value will determine what percentage of browser clients are randomly selected to have the grace period disabled for a particular site. These cohort selections will persist for a client, up until they reinstall the browser or clear site storage, or the site updates its percentage value again.

Site developers will also have a mechanism for inspecting the rollout of their opt-out percentage value config, dependent on browser implementation. In third-party use cases where both the top-level and embedded site are rolling out their tokens, the maximum of their percentage values is used.

## Key Use Cases
The following flows detail possible development journeys for sites incorporating a replacement for the grace period.

### Use Case 1: Top-level deprecation trial
1. Site developer ExampleTop detects breakage due to third-party cookie turndown on their site www.example.com.
1. ExampleTop follows the [documentation](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/first-party-deprecation-trial) for applying for the deprecation trial. They apply for the site pattern [*.]example.com to grant all subdomains the ability to share the deprecation trial token.
1. After internal review by Chrome, their application is approved. [*.]example.com is added to the grace period and the user-facing breakage is mitigated within 48 hours.
1. ExampleTop integrates the DT token to the request header for all affected cookies on [*.]example.com, and performs local testing.
1. ExampleTop creates the .well-known resource with an initial FirstPartyOptOutPercentage value of 10, and hosts it on example.com/.well-known/tpcd/grace-period.json. It gradually increments FirstPartyOptOutPercentage to 100% while monitoring breakage reports, and the grace period is fully disabled.
1. Chrome will detect that the grace period has been fully ramped and disable it by default for the site. At this point, ExampleTop can stop serving grace-period.json.
1. In the long run, ExampleTop migrates off of the tokens to one of the [long-term privacy-preserving APIs](https://developers.google.com/privacy-sandbox/3pcd/prepare/prepare-for-phaseout#take_action_to_ensure_your_sites_are_prepared) offered by the browser.

### Use Case 2: Embedded site deprecation trial
1. Site developer ExampleEmbed detects breakage due to third-party cookie turndown on their third-party plugin embed.example.com.
1. ExampleEmbed follows the [documentation](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/third-party-deprecation-trial) for applying for the embedded site deprecation trial. They apply for the site pattern embed.example.com to grant only the exact subdomain embed.example.com the ability to use the deprecation trial token.
1. After internal review by Chrome, their application is approved. embed.example.com is added to the grace period and the user-facing breakage is mitigated within 48 hours.
1. ExampleEmbed creates the .well-known resource with an initial ThirdPartyOptOutPercentage value of 0, and hosts it on embed.example.com/.well-known/tpcd/grace-period.json.
1. ExampleEmbed migrates their third-party cookies to instead use [Partitioned Storage](https://developers.google.com/privacy-sandbox/3pcd/chips), a long-term privacy-preserving API, and performs local testing.
1. ExampleEmbed increments ThirdPartyOptOutPercentage to 10. However, they notice an increase in user-facing breakage and identify a bug in their CHIPS adoption. They revert ThirdPartyOptOutPercentage to 0 while the fix is rolled out. The breakage is mitigated within 48 hours of the resource update.
1. Once the fix is released and more local validation is added, ExampleEmbed restarts the rampup of ThirdPartyOptOutPercentage to 100%.
1. Chrome will detect that the grace period has been fully ramped and disable it by default for the site. At this point, ExampleEmbed can stop serving grace-period.json.

## Considered Alternatives
Possible alternatives to the proposal above include:

1. Extend the current grace period with a “opt-out percentage” flag in the internal browser configuration. This adds support for a staged opt-out, but it’s not self-service and would require close and ongoing alignment between the developer and Chrome, which doesn’t scale well. This alignment also increases the SLA duration for rolling out updates, including rollbacks in case of breakage.

1. Design an [origin trial](https://www.chromium.org/blink/origin-trials/) which clients apply for to receive a “re-enablement trial” cookie token. This comes with the same downsides of the current deprecation trial - sites have no mechanism for testing the tokens in production without reaching out to Chrome.

1. Define a well-known file as specified above, but have each client fetch the resource, rather than Chrome. This decreases the latency for opt-out percentage value updates, and potentially gives the site more control over its opt-out populations. However, it has significant downsides:
- It would expose client browsing history via its network requests to specific .well-known resources.
- It would require requests to the domain of embedded sites (if there is a third-party grace period active) which adds new cross-site information leakage through timing attacks, etc.
- It would greatly increase the traffic load to the .well-known resource and could overload its server.
- It would add a performance cost due to an additional request for each client navigation, which could slow down the browser.

## Privacy and Security Considerations
Although there aren't inherent security risks with fetching .well-known resources from server-side infrastructure, there is a potential vulnerability of fetching bad data which could be used to exploit Google’s servers. We will mitigate this by rate-limiting HTTP requests, by scoping the requests to only the specific .well-known file, and by not storing any irrelevant or invalid information from the file - only what matches the [spec](https://github.com/explainers-by-googlers/3pcd-deprecation-trial-staged-rollout/blob/main/well-known-specification.md).

Assigning every user to a randomized cohort for each site in the grace period creates a stable identifier with a high level of entropy for each client, which could be a privacy concern if it is used to identify distinct users. We are mitigating this risk factor by ensuring these cohorts are cleared on site data deletion. These cohorts are also independent and only apply to one site, so it’s not possible for a site to join the cohorts and construct a cross-site identifier for tracking.

We are also defining an internal override flag for each site on the grace period. This would be set only in cases where the site developer is unable to edit the site’s .well-known resource, or if there is a broader issue with the system that fetches and updates the resources. Access to this override flag will be strictly gated using a two-party ACL system.

## Stakeholder Feedback / Opposition
TBD - This proposal is pending feedback from the appropriate WGs.

## References & acknowledgements
Thank you for design input, feedback, and review from:\
Ben Kelly\
Artur Janc\
Christian Dullweber
