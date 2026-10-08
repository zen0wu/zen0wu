# Architecture examples

Read the section relevant to the current decision. Product-specific statements illustrate the reasoning and should be reverified before use as current technical guidance.

- [Layered integrations](#layered-integrations)
- [Infrastructure layers](#infrastructure-layers)

## Layered integrations

Bad:

> The platform supports inline comments, but the provider does not, so the current path needs a separate webhook.

Why?

- "Platform", "provider", "current path", and "separate webhook" are not defined.
- The reader cannot tell whether "provider" means the external product's native integration, an API client, or an infrastructure-as-code adapter.
- The same component may be renamed later as a "service" or "built-in behavior", making the explanation internally inconsistent.
- The claim mixes product capability, configuration exposure, and the repository's current configuration.

Good:

> GitHub emits a `pull_request_review_comment` event for each new inline comment or reply. Buildkite's native GitHub integration can create builds from that event, but requires the comment to contain a configured command phrase and come from a trusted author. The Buildkite Terraform provider does not expose the setting that enables this trigger. Therefore, the current Terraform configuration cannot enable arbitrary inline-comment triggers.

Why?

- Each component has one stable name.
- Every capability and limitation is attributed to the component responsible for it.
- The explanation defines the event and distinguishes product behavior, Terraform exposure, and current configuration.
- The conclusion follows from the named constraints without introducing another vague mechanism.

## Infrastructure layers

Bad:

> Non-Auto EKS requires one node group per AZ for persistent EBS.

Why?

- "Non-Auto EKS" does not identify the node provisioner.
- "Node group" conflates EKS Managed Node Groups with Karpenter NodePools.
- It presents a limitation of multi-AZ Auto Scaling Groups as a limitation of standard EKS.
- It calls something impossible without naming the constraints under which it is impossible.

Good:

> With an EKS Managed Node Group backed by a multi-AZ Auto Scaling Group, replacement placement is not reliably driven by a pending Pod's EBS AZ constraint. Standard EKS with upstream Karpenter can instead use one multi-AZ NodePool: Karpenter reads the existing PersistentVolume's zone affinity and provisions the replacement node in that AZ. EKS Auto Mode can do the same while operating the node provisioning and EBS integration for us.

Why?

- It names the responsible mechanisms and resources.
- It scopes the limitation to the configuration where it applies.
- It distinguishes platform capability from the current implementation.
- It identifies the nearest valid alternative.
