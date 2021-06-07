---
title: "Kerckhoffs’s Law for Security Engineers"
description: What can we learn from Kerckhoffs's law for all security engineering
date: 2021-06-04
tags:
  - cryptography
  - security engineering
layout: layouts/post.njk
---

One of the first lessons in cryptography 101 is [Kerckhoffs’s law](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle): a cryptosystem should be secure even if everything about the system, except the key, is public knowledge. This is an often-repeated maxim accompanied with “there is no security with obscurity.”

I always found this framing confusing: it felt inconsistent within itself. “*don’t rely on secrecy except for the secrecy of the key*” What is so special about keys? Why is it ok to rely on the secrecy of keys and not on secrecy of anything else? And because it is so focused on keys, it’s hard to really take this foundational lesson and apply it in contexts other than cryptographic algorithms.

In [Schneier’s framing](https://www.theatlantic.com/magazine/archive/2002/09/homeland-insecurity/302575/), the aim here is *resilience*. Secrecy is a source of brittleness. When something you relied on being secret inevitably leaks, it can be the cause of catastrophic failure. I.e., if you rely on your cryptographic algorithm remaining secret, one day it’s not and suddenly you are in a world of pain. 

This provides a framing of Kerckhoffs’s law that I have found very useful: the security of your system should ***rely only on secrecy of things you can change easily***. Just using open, well-known algorithms for encryption is not enough. You need to be able to change the key easily.

Usually, this means *supporting* key rotation in the design—e.g., a encryption scheme that integrates with your key management service also includes the key version in the ciphertext. But, in my experience, a stated design goal to support rotation rarely works. Actually rotating the key regularly is the only way to be sure that you can change the key easily. Ideally, your key management service will automatically rotate secrets at a cadence.

This is then an easy pattern to look for in security design or review: a cryptosystem where the key is static or not rotated regularly is a source of brittleness almost as bad as having a secret algorithm. Unfortunately, the common framing of Kerckhoffs’s law has meant every engineer will detect and flag the use of an obscure, homebrew crypto algorithm but will happily stamp an encryption scheme that never rotates the key (or worse, *can’t* rotate the key).

The reframing also provides a useful lesson for security engineering: relying on secret you are not rotating regularly is a security risk. As much as possible, rely on short-lived secrets and minimize long-lived secrets. For example, for AWS access, do not rely on IAM access keys and secrets; prefer the short-lived credentials provided by role assumption. Ask your vendors to integrate with you using role assumption rather than IAM access keys (something [even Amazon pushes for](https://aws.amazon.com/blogs/apn/securely-accessing-customer-aws-accounts-with-cross-account-iam-roles/)).

I find this philosophy also useful organizationally! A security team unwilling to talk about the architecture or design with customers or researchers is an anti-pattern: the security architecture of an application is not something you can change easily, so sharing it and getting more eyes on it can only help you make secure! Openness and transparency is the mark of a leading security organization. For example, see the Uber security team’s [open treasure map](https://eng.uber.com/bug-bounty-map/). Or, see the Gitlab security team’s goal of being the [most open security team in the world](https://about.gitlab.com/handbook/engineering/security/). Or, just the [Chrome Security team’s approach](https://www.chromium.org/Home/chromium-security) overall.

Finally, it’s useful to remember the real goal here is robust security. Agility is the only way to achieve true robustness, as we learn of new vulnerabilities in systems and algorithms. Just openness in and of itself does not magically make things secure. Rather, openness is a bet that something has had more eyes on it. But even the most reviewed systems fail over time. TLS1.0, RC4, MD5, CBC, RSA — some of the most heavily reviewed systems have been broken over time and the ones we use today will likely be broken some day (hopefully, far) in the future. Can your system adapt?

*Thanks to [Clint Gibler](https://tldrsec.com/), Hongyi Hu, Matthew Finifter, Max Burkhardt, Patrick Toomey, Russ Allbery for their feedback. All mistakes are mine and I would love your feedback!*
