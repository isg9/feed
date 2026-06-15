---
title: Ruby's Bundler adds a cooldown feature
url: https://lwn.net/Articles/1076526/
published: "2026-06-05T12:57:00Z"
feed: lwn
guid: https://lwn.net/Articles/1076526/
---

# Ruby's Bundler adds a cooldown feature

[Version
4.0.13](https://blog.rubygems.org/2026/06/03/4.0.13-released.html) of Ruby's [Bundler](https://bundler.io/) package-manager has [added
dependency cooldowns](https://blog.rubygems.org/2026/06/03/cooldown-let-new-gems-be-vetted.html) in order to help mitigate the effect of supply-chain attacks:

> Most supply-chain attacks against RubyGems exploit a narrow window: an account is compromised, a malicious version ships, and any `bundle install` in the minutes that follow resolves straight to it. Bundler 4.0.13 introduces cooldown, a time-based filter that refuses to resolve to a version until it has been public for at least *N* days. Releases too new to have been scrutinized are passed over in favor of ones that have aged past the window.
>
> The feature was [designed in
> the open](https://github.com/ruby/rubygems/discussions/9113), drawing on [how
> other ecosystems approach the same problem](https://dev.to/hsbt/should-rubygemsbundler-have-a-cooldown-feature-40cp). It is opt-in, and complements rather than replaces existing defenses like mandatory 2FA and trusted publishing.

LWN [covered](https://lwn.net/Articles/1068692/) dependency cooldowns in April, and the [takeover of RubyGems and
Bundler](https://lwn.net/Articles/1040778/) in October 2025.
