# ​​The "tpcd/grace-period.json" Well-Known Resource Identifier

This page describes the use and content of URIs with the path /.well-known/tpcd/grace-period.json.

## Use
A site currently in the [third-party cookies Deprecation Trial](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/third-party-deprecation-trial) grace period might express interest in a staged rollout of the long-term behavior (Deprecation Trial tokens or a privacy-preserving API). Such sites can host this well-known resource to opt out of the grace period for a percentage of clients, in order to avert widespread, unintended breakage on end-user experience if there are issues with the intended replacement.

## Content
The /.well-known/tpcd/grace-period.json resource must follow the [JSON schema](https://json-schema.org/) stated below:

```
{
  "type": "object",
  "anyOf": [
    {
      "required": ["FirstPartyOptOutPercentage"]
    },
    {
      "required": ["ThirdPartyOptOutPercentage"]
    },
    {
      "required": ["SubdomainOptOuts"]
    }
  ],
  "properties": {
    "FirstPartyOptOutPercentage": {
      "type": "integer",
      "enum": [0, 25, 50, 100]
    },
    "ThirdPartyOptOutPercentage": {
      "type": "integer",
      "enum": [0, 25, 50, 100]
    },
    "SubdomainOptOuts": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "Subdomain": {
            "type": "string"
          },
          "MatchSubdomains": {
            "type": "boolean"
          },
          "FirstPartyOptOutPercentage": {
            "type": "integer",
            "enum": [0, 25, 50, 100]
          },
          "ThirdPartyOptOutPercentage": {
            "type": "integer",
            "enum": [0, 25, 50, 100]
          }
        }
      }
    }
  }
}
```

"FirstPartyOptOutPercentage" denotes the site's opt-out value when it is in a top-level context embedding third-party cookies. For instance, if site example.com is enrolled in the first-party grace period, the `FirstPartyOptOutPercentage` on `example.com/.well-known/tpcd/grace-period.json` represents what % of Chrome clients opt out of the grace period and have third-party cookies blocked when embedded on example.com.

"ThirdPartyOptOutPercentage" denotes the site's opt-out value when it is using third-party cookies in an embedded context. For instance, if site example.com is enrolled in the third-party grace period, the `ThirdPartyOptOutPercentage` on `example.com/.well-known/tpcd/grace-period.json` represents what % of Chrome clients opt out of the grace period and have third-party cookies blocked when a cross-site host embeds an example.com iframe.

"SubdomainOptOuts" allows specifying distinct opt-out values for subdomains of the current site. For instance, if site example.com is enrolled in the first-party grace period, and this is the resource defined at `example.com/.well-known/tpcd/grace-period.json`:

```
"FirstPartyOptOutPercentage": 25,
"ThirdPartyOptOutPercentage": 25,
"SubdomainOptOuts": [
  {
    "Subdomain": "sub1.example.com",
    "MatchSubdomains": true,
    "FirstPartyOptOutPercentage": 100,
    "ThirdPartyOptOutPercentage": 50
  },
  {
    "Subdomain": "sub2.example.com",
    "MatchSubdomains": false,
    "FirstPartyOptOutPercentage": 0,
    "ThirdPartyOptOutPercentage": 0
  }
]
```

Then:
* 25% of eligible clients have third-party cookies blocked when embedded on example.com.
* 100% of eligible clients have third-party cookies blocked when embedded on sub1.example.com.
* 100% of eligible clients have third-party cookies blocked when embedded on sub.sub1.example.com.
* 0% of eligible clients have third-party cookies blocked when embedded on sub2.example.com.
* 25% of eligible clients have third-party cookies blocked when embedded on sub.sub2.example.com.

The `grace-period.json` file must be well formed in order to take effect. To avoid errors, check your file content with the [grace period opt-out validation tool](https://3pcd-mitigations-wrv.glitch.me/).
