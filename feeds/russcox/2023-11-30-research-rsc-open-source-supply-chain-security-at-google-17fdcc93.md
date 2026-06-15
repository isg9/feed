---
title: 'research!rsc: Open Source Supply Chain Security at Google'
url: https://research.swtch.com/acmscored
published: "2023-11-30T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/acmscored
---

# research!rsc: Open Source Supply Chain Security at Google

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Open Source Supply Chain Security at Google Russ Cox November 30, 2023 *research.swtch.com/acmscored* Posted on Thursday, November 30, 2023.

I was a remote opening keynote speaker at [ACM SCORED 2023](https://scored.dev/), which we decided meant that I sent a video to play and I was on Discord during the talk for attendees to text directly with question as the video played, and then we did some live but still remote Q&A after the talk.

My talk was titled “Open Source Supply Chain Security at Google” and was 45 minutes long. I spent a while at the start defining open source supply chain security and a while at the end on comparisons with the 1970s. In between, I talked about various supply chain-related efforts at Google. All the Google efforts mentioned in the talk have been publicly discussed elsewhere and are linked below.

Here are the [talk video](https://youtu.be/6H-V-0oQvCA) and [talk slides](acmscored.pdf). Opinions expressed in the talk about languages and the last half century of supply chain security are mine, not Google’s.

References or acknowledgements for the slides:

- Crypto AG: [Guardian](https://www.theguardian.com/us-news/2020/feb/11/crypto-ag-cia-bnd-germany-intelligence-report) and [Washington Post](https://www.washingtonpost.com/graphics/2020/world/national-security/cia-crypto-encryption-machines-espionage/)
- Enigma photograph: personal photo, taken at Bletchley Park in 2012

- XcodeGhost: [Palo Alto Networks](https://unit42.paloaltonetworks.com/novel-malware-xcodeghost-modifies-xcode-infects-apple-ios-apps-and-hits-app-store/)
- Juniper Attack: [CACM](https://cacm.acm.org/magazines/2018/11/232227-where-did-i-leave-my-keys/fulltext), [Eprint](https://eprint.iacr.org/2016/376.pdf), [Bloomberg](https://www.bloomberg.com/news/features/2021-09-02/juniper-mystery-attacks-traced-to-pentagon-role-and-chinese-hackers)
- SolarWinds: [Wired (Kim Zetter)](https://www.wired.com/story/the-untold-story-of-solarwinds-the-boldest-supply-chain-hack-ever/)
- NPM event-stream: [Ars Technica](https://arstechnica.com/information-technology/2018/11/hacker-backdoors-widely-used-open-source-software-to-steal-bitcoin/), [NPM](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident)
- iMessage JBIG2: [Project Zero](https://googleprojectzero.blogspot.com/2021/12/a-deep-dive-into-nso-zero-click.html)
- Log4j: [Minecraft](https://www.minecraft.net/en-us/article/important-message--security-vulnerability-java-edition), [CISA](https://www.cisa.gov/news-events/news/apache-log4j-vulnerability-guidance)
- [Kubernetes on Open Source Insights](https://deps.dev/go/k8s.io%2Fkubernetes/v1.28.4/dependencies/graph) and [comparing versions](https://deps.dev/go/k8s.io%2Fkubernetes/v1.28.4/compare?v2=v1.29.0-rc.0)
- [Sigstore](https://www.sigstore.dev/)
- “ [Perfectly Reproducible, Verified Go Toolchains](https://go.dev/blog/rebuild)”

- “ [How Go Mitigates Supply Chain Attacks](https://go.dev/blog/supply-chain)”

- Two-person photograph: [Air Force National Museum](https://www.nationalmuseum.af.mil/Visit/Museum-Exhibits/Fact-Sheets/Display/Article/197675/launching-missiles/), public domain

- [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev/spec/v1.0/levels)
- [Security Scorecards](https://securityscorecards.dev/)
- Capslock: [blog post](https://security.googleblog.com/2023/09/capslock-what-is-your-code-really.html), [repository](https://github.com/google/capslock)
- Google [Open Source Security Rewards](https://bughunters.google.com/open-source-security)
- Google Project Zero: [blog post](https://security.googleblog.com/2014/07/announcing-project-zero.html), [excellent video](https://www.youtube.com/watch?v=My_13FXODdU)
- OSS-Fuzz: [blog post](https://opensource.googleblog.com/2016/12/announcing-oss-fuzz-continuous-fuzzing.html), [repository](https://github.com/google/oss-fuzz)
- [Syzkaller dashboard](https://syzkaller.appspot.com/upstream)
- Internet worm: [New York Times](https://timesmachine.nytimes.com/timesmachine/1988/11/04/issue.html)
- [NSA Software Memory Safety](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/0/CSI_SOFTWARE_MEMORY_SAFETY.PDF)
- [Go home page](https://go.dev/)
- [Rust home page](https://www.rust-lang.org/)
- SBOMs: “ [NTIA: Framing Software Component Transparency](https://www.ntia.gov/sites/default/files/publications/ntia_sbom_framing_2nd_edition_20211021_0.pdf)”, “ [CISA: Software Identification Ecosystem Option Analysis](https://www.cisa.gov/sites/default/files/2023-10/Software-Identification-Ecosystem-Option-Analysis-508c.pdf)”

- [Open Source Vulnerability](https://osv.dev) database

- Govulncheck: [blog post](https://go.dev/blog/govulncheck), [package docs](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck), [tutorial](https://go.dev/doc/tutorial/govulncheck)
- [Google Cloud Artifact Analysis](https://cloud.google.com/artifact-registry/docs/analysis)
- [Air Force Review of Multics](https://seclab.cs.ucdavis.edu/projects/history/papers/karg74.pdf) (quotes are from pages numbered 51 and 52 on the paper, aka PDF pages 55 and 56)

- Thompson backdoor: “ [Reflections on Trusting Trust](https://dl.acm.org/doi/10.1145/358198.358210)” (1983) and [annotated code](https://research.swtch.com/nih) (2023)
