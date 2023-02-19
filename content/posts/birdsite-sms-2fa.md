---
title: Twitter ditches SMS as 2FA for non-paying users
date: "2023-02-19"
description: "Commentary about Twitter's decision to only allow SMS as 2FA for Twitter Blue customers"
tags: ["Technology"]
---

Effective next month, Twitter will disable SMS-based two-factor authentication (2FA) for users who have not paid Musk's $8 fee for Twitter Blue.

<figure>
  <a href="musk-announcement.png" data-lightbox="musk-announcement"><img src="musk-announcement.png" class="blog-img img-full"></a>
  <figcaption>Source: <a href="https://twitter.com/elonmusk/status/1627059645293670401">Twitter</a></figcaption>
</figure>

Musk claims that Twitter is spending $60M on fraudulent SMS challenges from a longstanding practice known as [Toll Fraud](https://twitter.com/EricAschner/status/1626999229401948163), where bad-acting telcos who charge high rates for delivering SMS messages surreptitiously use bots to force generation such SMS verification messages to inflate their bills. While Toll Fraud is undoubtedly a real thing that I have no doubt Twitter wastes a lot of money on, I won't take the $60M figure at face value because the fact remains that [most 2FA challenges are probably malicious](https://twitter.com/pookleblinky/status/1627173306372587520) because that's 2FA working as designed. I'm more likely to believe Musk is lumping all SMS challenges sent that don't result in a successful login as problematic fraud.

Compared to other 2FA mechanisms like TOTP, SMS is comparatively insecure; it's far easier to intercept an SMS message than it is to get access to Google Authenticator. Many on Twitter are responding that [this decision is good for security](https://twitter.com/mjackson/status/1626999765904031744) because you shouldn't be using SMS for 2FA anyway.

This argument isn't convincing for me.

First, for most of us on Twitter, SMS is sufficiently secure as a second factor. For a system to be sufficiently secure, the cost of bypassing the security controls must exceed the value that can be extracted by doing so. If I store $50 in a safe that it would cost $5000 to break into, I can rest assured that money is generally safe. That's why password-only authentication on Twitter is insufficient, even for the majority of us: it's low cost to use hacked databases of passwords to mass compromise Twitter accounts and use them to pump crypto scams, to run a Zelle scam, or fo any other number of profitable scams. However the cost of compromising most users' SMS vastly exceeds that value. Unless you're a (legacy) blue checkmark or have real influence in the world, SMS provides sufficient security.

What's likely to happen is that by raising the complexity of supporting 2FA - including requiring the installation of Google Authenticator or even learning about other 2FA technologies like TOTP or Yubikey - many users will simply disable 2FA altogether.

Second, Twitter's operating on a theory is that the $8/mo cost is enough to make the cost to unscrupulous telcos running Toll Fraud programs sufficiently high as to make generating them unprofitable. That theory is certainly mistaken. A realistic example: some such bad acting telcos charge of $0.05 per SMS message, and might pay a scammer 20% of that in order to automate the mass generation of such messages. Stolen credit card numbers are cheap enough to acquire on the dark web. The scammer would only have to generate 800 SMS challenges per account in a month to break a profit, which works out to about one challenge per hour. Automated systems can generate them as often as Twitter will let them, which is several times higher than that.

Finally, if security is the aim, allowing people to pay money to avoid appropriate security controls is hardly consistent with that aim. It's disingenuous to suggest that Twitter's intention here is greater security for its user base.

Ultimately, to me this lands as yet another half-baked edict from Twitter's Peter Pan Syndrome stricken owner without showing the faintest interest in understanding the impact.