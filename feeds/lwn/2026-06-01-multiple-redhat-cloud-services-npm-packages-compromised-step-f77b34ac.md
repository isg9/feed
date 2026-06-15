---
title: Multiple redhat-cloud-services npm packages compromised (StepSecurity Blog)
url: https://lwn.net/Articles/1075742/
published: "2026-06-01T14:05:04Z"
feed: lwn
guid: https://lwn.net/Articles/1075742/
---

# Multiple redhat-cloud-services npm packages compromised (StepSecurity Blog)

StepSecurity is [reporting](https://www.stepsecurity.io/blog/multiple-redhat-cloud-services-npm-packages-compromised) that a number of npm packages in the `@redhat-cloud-services` scope include malware that runs automatically on every `npm install`:

> The payload is a multi-stage credential harvester that sweeps GitHub Actions secrets along with AWS, GCP, Azure, Kubernetes, HashiCorp Vault, npm, and CircleCI tokens, and it is purpose-built to evade detection, including an explicit attempt to bypass StepSecurity Harden-Runner.
>
> StepSecurity analyzed `@redhat-cloud-services/host-inventory-client@5.0.3` in full. Its `index.js`, executed at install time, is 4.2 MB, a file that should weigh a few kilobytes, with the real payload buried under three separate layers of obfuscation. The malware is also a self-propagating worm: using stolen npm tokens and npm's `bypass_2fa` parameter, it republishes backdoored versions of other packages on its own, even against accounts protected by two-factor authentication, so every infected machine can seed the next wave with no attacker involvement. All affected packages were published via GitHub Actions OIDC from the `RedHatInsights/javascript-clients` repository, indicating the upstream CI/CD pipeline itself was compromised. Analysis of the remaining packages is ongoing.

A [blog
post](https://safedep.io/redhat-cloud-services-hit-by-mini-shai-hulud-npm-worm/) from SafeDep has additional analysis about the incident. We did not find an advisory from Red Hat on this yet.
