# ​​The "tpcd/grace-period.json" Well-Known Resource Identifier

This page describes the use and content of URIs with the path /.well-known/tpcd/grace-period.json.

## Use
A site currently in the [third-party cookies Deprecation Trial](https://developers.google.com/privacy-sandbox/3pcd/temporary-exceptions/third-party-deprecation-trial) grace period might express interest in a staged rollout of the long-term behavior (Deprecation Trial tokens or a privacy-preserving API). Such sites can host this well-known resource to opt out of the grace period for a percentage of clients, in order to avert widespread, unintended breakage on end-user experience if there are issues with the intended replacement.

## Content
The /.well-known/tpcd/grace-period.json resource must follow the schema stated below:

```
{
  "type": "object",
  "required": ["FirstPartyOptOutPercentage", "ThirdPartyOptOutPercentage"],
  "properties": {
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
```