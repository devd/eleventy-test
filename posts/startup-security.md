---
title: "Early Security for Startups"
description: What should a startup without a security team do for security? 
date: 2022-03-28
tags:
  - security engineering
  - startups
layout: layouts/post.njk
---

*I once read blogging advice:* *if someone asks you something, write it as a blog post and soon you will have a blog. One advice I am often asked is “I am a new startup; how should I approach security?” Here goes my attempt, based on* [*my experience*](https://linkedin.com/in/devdattaakhawe) *working at and advising hypergrowth companies. Please send me your thoughts and feedback!*


# Who is this for?

This is advice for early-stage startups without security teams (the post ends with advice on your first security hire!). It focuses on the basics for an early-stage startup. If you already have a security team, listen to the team!
 
Most of my experience has been in product-led enterprise SaaS businesses. This guide is likely useful to such startups. If you are working on things like social network or a crypto product, your threat model and priorities are very different, and this guide is likely not correct for you. That said, it might still be useful!

The guide also assumes that the founders and early members of the team already care about security. As an example, if you are not already using HTTPS everywhere, if you are not already on a framework like Rails that auto-escapes SQL queries, and so on, this guide is likely not for you.

This guide is also *prescriptive*: in addition to broad advice, it specifies tools and vendors to use, as founders have found this very useful. I make no attempt to be unbiased here: these are products I like and/or have some form of connection or interest in.

# What is security?

An easy way to come up with what a team without a security team should do is to start with understanding what a security team, if present, would do. I think of a security team’s role in 3 categories ([inspired by](https://about.gitlab.com/handbook/engineering/security/) [GitLab’s docs](https://about.gitlab.com/handbook/engineering/security/)): 

1. Protect the company from data breaches:  Security teams are primarily responsible for preventing unauthorized access to data, broadly construed. A security team will help contextualize and prioritize security risk to the company and manage mitigations.
2. Secure the product:  a software company ships products, and a security team will often be ensuring that these products do not have technical vulnerabilities (typically, as part of the product security sub-function).
3. Customer Assurance: It’s not enough for you and your products to be secure; your customers also need to trust you are secure. This involves things like compliance certifications, answering customer security questionnaires, and helping your customers use your product securely.

Let’s discuss each category in detail, especially in the context of an early-stage startup.

# Protect the company from data breaches

The *possibility* of data breaches is everywhere and the only way to prevent all data breaches is stop doing anything. Instead, a security team focuses on *risk reduction.* A security team’s primary job is to a) identify and prioritize data breach risks to the business and b) manage/mitigate them through technology and/or processes. 

## What is risk?

Risk is usually defined as the product of probability and impact: e.g., high probability and high impact issues are the highest risk. While the math helps, I find it easiest to explain risk with a story.

I love hiking and I once hiked up Mt. Kilimanjaro. When I mention hiking Kili, people often ask me if the mountain was scary/risky. Truth is, mountain was fun, exhilarating, and probably one of the healthiest things I did that year. Statistically, the riskiest part of the trip was the car ride to the mountain and back.  

Security for early startups is similar: while you are doing something unique, most of your risk is due to the everyday hazards that affects everyone. Wear your seatbelt and you will be fine. 

Furthermore, for most startups, the biggest business risk is irrelevance: you don’t get to product market fit and/or you don’t cross the chasm into wide adoption. Hackers are not targeting startups with no users. But there is a “background radiation” of malicious activity on the internet targeting every modern software company that everyone needs to defend against. This background or latent risk is behind most breaches that you hear about. It is rare and exceptional for a startup to be specifically targeted persistently; rather, they get compromised by opportunistic criminals. 


## Security Risks for a startup

**If you take** ***one thing*** **away from this doc**, it is to focus on the common source of breaches. While the security and compliance of your product itself is important, what will get you hacked is what gets everyone else. That latest complicated side-channel exploit you read on HackerNews is unlikely to affect your startup.

If I am asked what the most common source of large breaches in the last few years are, it's 

1. Ransomware 
2. Cloud misconfiguration/leak
3. Credential compromise via phishing, password reuse, etc.

Let's go through mitigating these risks at a startup in turn.

## Ransomware

For the typical startup, ransomware is unlikely to be a large risk since most startups have small teams running modern OSes like MacOS with no legacy code. While this can change (MacOS malware is growing!), I do not believe for most startups, MacOS ransomware is an immediate risk worth prioritizing.

## Cloud Misconfiguration

While modern cloud infrastructure is amazing, it’s far too easy to make a mistake and introduce a serious, massive vulnerability that opportunistic criminals will latch on to. [Christophe has a fantastic blog post](https://blog.christophetd.fr/cloud-security-breaches-and-vulnerabilities-2021-in-review/) looking back at breaches in 2021: the most common source of breaches are a) static credentials, b) misconfigured data stores that are publicly readable, and  c) SSRF on metadata services.

Since AWS continues to be the most common cloud provider, I am going to focus on my recommendations for AWS that mitigate the issues above:

- All access in AWS should be through short lived tokens based on role assumption, ideally through your SSO provider (below). Avoid all long-lived credentials (all tooling should support role assumption today; there is almost no reason to have long-lived IAM access keys).
- [Block public access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html) to S3 buckets; I also recommend [using modern tooling like SSM](https://www.figma.com/blog/inside-figma-getting-out-of-the-secure-shell/) so that no administrative ports (like the ssh port) are listening on the open internet.
- [Disable IMDSv1](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-options.html) for existing and all new machines ([terraform config](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#metadata-options))
- Regularly run a free cloud scanning tool ([cloudmapper](https://github.com/duo-labs/cloudmapper)/[scoutsuite](https://github.com/nccgroup/ScoutSuite) for AWS or [simplecspm for GCP](https://simplecspm.com/)) and check no AWS resources are public on the internet.

Most of these guidelines are from Scott’s [fantastic AWS security roadmap](https://summitroute.com/downloads/aws_security_maturity_roadmap-Summit_Route.pdf) freely available for all to use. Similar guides exist for [GCP](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations) and [Azure](https://www.coffeehousecoders.org/blog/azure_security_roadmap.html) (or, Marco has a [generic one](https://roadmap.cloudsecdocs.com/)). 


## Credential Stuffing

The next most common source of breaches is due to account takeover/compromise by phishing or [credential stuffing](https://en.wikipedia.org/wiki/Credential_stuffing) of employees. This is where an employee reuses a password for work and a personal account, and the latter gets hacked. Or an employee is phished. 

By far, the most effective security against this is using a strong, centralized service for login to all applications that enforces [phish-proof webauthn MFA](https://blog.cloudflare.com/cloudflare-now-supports-security-keys-with-web-authentication-webauthn/#:~:text=But%20where%20WebAuthn%20really%20shines%20is%20its%20particular%20resistance%20to%20phishing%20attacks.) as the only supported MFA mechanism. Start by requiring all SaaS applications (e.g., Datadog, Github, Zendesk, etc.)  to only allow logins through your identity provider (Google or Okta, typically), instead of using username/password. Next, in your identity provider settings, turn on MFA and enable the options so that only webauthn is supported. Note that every other form of MFA is vulnerable to phishing and webauthn is the only secure MFA mechanism. While a few years ago just MFA was sufficient mitigation from attacks, criminals today have learnt how to phish code-based MFA and/or compromise phone numbers to steal SMS codes. 

In Google Workspaces (the most common identity provider at early startups), this is trivial by [turning on Advanced Protection](https://support.google.com/a/answer/9378686?hl=en). Advanced Protection also enables a few other security settings that can be disruptive—in that case, you can enable the [“Only Security Key” option](https://support.google.com/a/answer/9176657) in the Google MFA settings for your organization.

Enforcing SSO on all applications can be expensive, especially for an early, bootstrapped startup. If you can’t enable SSO, at the minimum use Google sign-in wherever available and use unique, strong passwords on each application (via a password manager) and turn on MFA. Chrome’s inbuilt password manager is free but often doesn’t work as well; I recommend [1Password](https://1password.com/business/). 

## Other Common Risks

Finally, another class of issues is the Magecart-style [“web skimming”](https://en.wikipedia.org/wiki/Web_skimming#Magecart) attacks: this is where an attacker compromises third-party JS on your payment page and steals credit card numbers (e.g., [British Airways](https://techcrunch.com/2018/09/11/british-airways-breach-caused-by-credit-card-skimming-malware-researchers-say/)). Before ransomware became such a reliable, popular way for criminals to make money, web-skimming attacks were all the rage. While a security team can implement robust defenses later, I recommend only loading JS code from your own servers as much as possible. Note that [cache partitioning](https://developer.chrome.com/blog/http-cache-partitioning/) means that loading from a CDN provides limited performance benefit. If you really need to include a third-party hosted JS (like marketing, sales tooling), try [using privilege separation to limit blast radius](https://dropbox.tech/security/csp-third-party-integrations-and-privilege-separation). 

# Secure the product

A software company’s most important job is to ship great products that delight their customers. A good security team helps accelerate this by providing secure guardrails for product development and provides a second pair of eyes on the security risk of new features. Even with organizations with mature security teams, the responsibility for securing a feature lies with product teams: a security team only helps as an additional layer of accountability. 

For an early startup, the absence of a security team means that this second pair of eyes is missing: the responsibility of securing new products falls upon the engineers at a company. But this is like other fundamentals like clean code, reliability, scalability etc.; engineers need to balance product iteration and quality and make reasoned tradeoffs. While this is important, *I will stress again*: for a relatively new startup, you are far and away more likely to get compromised by phishing and AWS misconfigurations than a nuanced bug in your application. Moreover, it is quite possible that you will pivot and *securing the wrong product has tremendous opportunity cost*. 


## Authentication and login

Authentication is the one thing that every product *must* implement securely. Mistakes can be [embarrassing](https://techcrunch.com/2011/06/20/dropbox-security-bug-made-passwords-optional-for-four-hours/). For modern products, I recommend relying on [Google’s Identity services](https://developers.google.com/identity/): they have fantastic usability and conversion rates. Google handles tricky issues like abuse detection, account compromise, lockout, recovery, etc. and the default Google libraries are secure by design. As a product, you will have to decide on a reasonable session timeout that works for you but note that long session timeouts (> 3 days) will complicate security for your customers. As an enterprise product, at some point you will likely also need to implement SAML/SCIM support. [Okta/Auth0](https://auth0.com/docs/authenticate/protocols/saml/saml-sso-integrations) is a relatively well-known provider that can SAML/SCIM for you—you can either use their managed services or their [libraries](https://github.com/auth0/). 

If you do implement password sign-in, make sure to require email verification, [store passwords securely](https://dropbox.tech/security/how-dropbox-securely-stores-your-passwords), and implement a negative test (i.e., test that the wrong password does not let you log in). The [OWASP ASVS page on authentication](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x11-V2-Authentication.md) has a long list of password authentication best practices ([the broader ASVS](https://github.com/OWASP/ASVS/tree/master/5.0/en) is a great resource to review for any other features you are building too).


## Application Security

While there is a long list of potential product security concerns, the good news is that modern frameworks and cloud infrastructure has made shipping secure software orders of magnitude easier than in the past. Modern frameworks like React, Rails, etc. default to secure, features like [Dependabot (free with Github)](https://docs.github.com/en/code-security/supply-chain-security/managing-vulnerabilities-in-your-projects-dependencies/about-dependabot-security-updates) and [ECR container scanning (free with AWS)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html) make it easy to monitor and patch all your dependencies. Tools like [r2c/semgrep (with its community rules)](https://semgrep.dev/pricing) provide a good baseline of security linting on your code (and can be integrated with a few clicks on Github). A high-quality scanner like Detectify can provide a separate external continuous scan of your application, [again with a few clicks and self-serve signup](https://detectify.com/pricing). I generally recommend turning these on—in addition to the security benefits, all these will also come in very handy for customer assurance, as part of your sales process (discussed below). 

In addition to these generic checks for all web applications, it's also worth taking some time to think about unique risks applicable to your application. Are you shipping an electron app? Then, spend some time with the [electron security checklist](https://www.electronjs.org/docs/latest/tutorial/security/). Does your product allow sharing via links? Read up on [](https://www.youtube.com/watch?v=6AsVUS79gLw)[security for short links](https://freedom-to-tinker.com/2016/04/14/gone-in-six-characters-short-urls-considered-harmful-for-cloud-services/). Are you [storing credentials](https://support-my.plaid.com/hc/en-us/articles/4410324401047-Does-Plaid-have-access-to-my-credentials-)? Carefully design the handling/storage of these credentials. For your product, evaluate what are unique, critical pieces of tech in your stack and take a couple of hours to look for the relevant project/tool’s security configuration advice (if you can’t find it, ask a security engineer you trust). A good pentesting firm (I am happy to connect you with some) will also help you with resources specific to your needs; but you are the best judge of what are the critical components of your tech stack.


## Penetration Testing

As you get a firmer idea of what you are building and want to do a large launch or enterprise sales, [a white-box pentest](https://en.wikipedia.org/wiki/Penetration_test) can also provide a detailed evaluation of your product’s security posture. Your enterprise customers will anyways demand a pentest and using it as an opportunity to further harden your product is a great idea. “White-box” just means that the security testing firm has full read access to your source code. 

With pentests, the usual rule applies: pick two from Good, Fast, Cheap. I recommend always going with good and picking one between fast and cheap. As a startup, opportunity cost is high. A bad pentest can waste your time on security “vulnerabilities” that aren’t real risks and not uncover actual security issues. It’s hard to know which firms provide high quality pentests from their websites: ask the firms for references or talk to other founders for recommendations. I am a fan of [Doyensec](https://doyensec.com/). Regardless of who you pick, aim to invest in a long-term relationship where you can ask them for help with small tests to large-scale full product audits. 

# Customer Assurance

It is not enough to write trustworthy software; it is also important that your customers feel they can trust it. Customer assurance as a function is a broad area with lots of nuances, but, again, there are three categories of work that every enterprise SaaS startup will have to typically tackle. 


## Compliance Certifications

Certifications help assure your customers that your organization ensures a minimum baseline of security and governance practices. The most common certification in this category is the SOC2, with startups typically going with a 3-month observation period before switching to one year. For most startups today, the easiest path to getting a SOC2 certification is through one of the numerous SOC2 automation vendors like [Vanta](https://www.vanta.com/). Most startups think about compliance once they are ready to sell to bigger companies, but, for early startups, I recommend reading through [Christina’s](https://twitter.com/christinacaci) excellent [Before SOC2](https://www.vanta.com/blog/five-principles-for-building-a-secure-product) post or Latacora’s [SOC2 Starting 7](https://latacora.singles/2020/03/12/the-soc-starting.html) to avoid a lot of pain later: most of the advice is generally good advice (and matches my advice from earlier in this document).

One common mistake I have seen founders make is treating SOC2 as a set of requirements they must follow and wasting a lot of time on things that don’t make sense. If you find yourself thinking that a control just doesn’t make sense to you or is too disruptive to your organization, you can likely change it. SOC2 isn’t prescriptive on *what* you do; rather, auditors help validate and certify that you do what you *say you do*. Finally, don’t lose sleep over a “finding”: it usually isn’t a big deal unless it is particularly egregious. Some of the biggest companies you know (e.g., AWS) have several findings in their SOC2.

## Security Questionnaires

Even from your earliest customers, you are likely to hit a security review requirement that involves answering security questionnaires. These questionnaires are [particularly hated](https://latacora.micro.blog/its-weird-to/) in the industry, but they will likely come up often. Companies like [SecurityPal](https://securitypalhq.com/) can help you outsource this problem, but you still will need to help SecurityPal with the first few questionnaires. 

Typical early startups will not be able to achieve full marks on robust questionnaires by large-scale enterprises, but that’s fine. Avoid the trap of questionnaire-driven product development. Instead, I recommend founders prioritize customers who understand you are a startup and want to partner with you as you mature your practices. 

Often, most of the work involved in passing security questionnaires is writing down reasonable, common-sense policies. This is easier than it seems: Gitlab has [all their policies](https://about.gitlab.com/handbook/engineering/security/#-resources) public under an MIT license available that you can adapt; or, vendors like [SecurityPal](https://securitypalhq.com/) can help you with a basic set of policies.

For early-stage startups, individual deals can be attractive enough that you fill out questionnaires for each deal. This can get expensive over time. Bigger organizations tend to prefill a number of free questionnaires like the [Google VSAQ](https://github.com/google/vsaq), the [CAIQ](https://cloudsecurityalliance.org/blog/2021/09/01/what-is-caiq/), and other vendor questionnaires like [Whistic](https://www.whistic.com/) and [CyberGRX](https://www.cybergrx.com/). This can save you time, especially for smaller deals; I generally recommend waiting for the first time one of your customers asks you to fill these out. Once you have filled one out, proactively include the filled-out questionnaire along with your SOC2 and other certifications as part of the security “package” your sales team provides to prospects. Based on how sensitive the reports are, you can require an NDA or just use GMail confidential mode to share PDFs.  


## Governance/Security *inside* your application

Finally, the third category of work in customer assurance involves giving customers control over their data and configuration in your application. Your app can be secure, and your practices audited, but if it is easy to misconfigure and cause a breach, it's an issue that can still negatively impact customer trust. 

Typically, features that help here are put in the “enterprise” tier: this includes things like [enforcing SSO login](https://slack.com/help/articles/203772216-SAML-single-sign-on), enforcing [sharing controls](https://help.dropbox.com/teams-admins/admin/manage-team-sharing) for outside-of-org sharing, [activity/audit log](https://help.dropbox.com/teams-admins/admin/view-activity) for all access and so on. While it’s impractical to have an answer for all these, I recommend founders spend some time thinking about how it would play out for their early customers. For example, you don’t have to offer logs as part of the product but assure customers that in case of a security incident on the customer’s side, you will work with them to filter out and share logs *you* have. This can be a great tactical step to unblock deals while you build out these features longer term.

# Starting the security team

With luck, you hit product market fit and start scaling. When and how should a company start to build out the security team? I agree with [Magoo’s advice](https://scrty.io/): you likely [don’t need a CSO](https://medium.com/starting-up-security/you-dont-need-a-chief-security-officer-3f8d1a76b924) as your first security hire. Instead, a focused hire on a particular sub-function (customer assurance, product security) can be far more effective and keep options open later.  

My advice then is to first wait till you feel like you need a security hire and then, using the discussion above, figure out which sub-function you are trying to solve for. Do you need someone to focus on customer assurance (e.g., compliance and sales security reviews)? Or do you need someone to help with the security of your cloud infrastructure?

What hiring in security *cannot* help with is engineering culture; culture is set by the founders and early engineers. An engineering team that values and cares for security will achieve far better outcomes than an engineering team that doesn’t care---hiring in security will not change that. Don’t wait to hire in security to build a culture of security (like any other horizontal function like performance).

Once you have identified the exact need you want to fill, I recommend posting the role as soon as possible. Security talent is in high demand! But take your time for the first hire: a *bad* first security hire can do more damage than a missing security team. Non-technical skills are particularly important for the first hire: avoid toxic individuals and only hire someone who believes in the mission of the company.
