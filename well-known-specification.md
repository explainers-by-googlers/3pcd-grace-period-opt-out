# ​​The "deprecation-trials/tpcd/rollout.json" Well-Known Resource Identifier

This page describes the use and content of URIs with the path /.well-known/deprecation-trials/tpcd/rollout.json.

## Use
A site currently under the Third-party cookies deprecation trial grace period might express interest in participating in a staged rollout of their deprecation trial token in order to avert widespread, unintended breakage on end-user experience. These breakage could originate from varied causes such as missed/wrong/partial configuration and other edge-cases. Such sites must host the well-known resource.

## Content
The /.well-known/deprecation-trials/tpcd/rollout.json resource must follow the schema stated below:

```
{
  "type": "object",
  "required": ["RolloutPercentage"],
  "properties": {
    "RolloutPercentage": {
      "type": "integer",
      "minimum": 0
      "maximum": 100
    }
  }
}
```
