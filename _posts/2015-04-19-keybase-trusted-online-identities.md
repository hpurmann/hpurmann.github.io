---
layout: post
title: Trusted online identities
category: "dev"
---

Imagine you could send encrypted text messages to people only knowing their social media username. Imagine you could verify a connection between someone's twitter account and their website.

This is both possible with [Keybase](https://keybase.io/). The service lets you link your public key to your online identities such as GitHub, Twitter, Reddit, Bitcoin or your personal website. Other users can then verify your accounts.

Have you heard of these crypto key signing *parties* where a few nerds meet, check each others identity cards and approve that they met this person and verified their key? That is basically the concept of the [Web of Trust](http://en.wikipedia.org/wiki/Web_of_trust). It lets you trust in the identity of some unknown person C because you know person B who verified C.

Keybase is not trying to replace the Web of Trust but enhance it with the approach of online identities instead of government documents.

From the [Keybase docs](https://keybase.io/docs/server_security/tracking):

> Keybase aims to provide public keys that can be trusted without any backchannel communication. If you need someone's public key, you should be able to get it, and know it's the right one, without talking to them in person.

Keybase is [on GitHub](https://github.com/keybase), they provide a nice [command line tool](https://keybase.io/docs/command_line) and implemented [PGP in JavaScript](https://keybase.io/kbpgp). Oh, and there is a [public API](https://keybase.io/docs/api/1.0) to integrate Keybase into your service. Currently, Keybase is still in closed alpha. You can only get in with an invitation.

As of this writing, I have a few invitations left. To get one, simply send me an encrypted mail explaining why you want to be in. My mail address? You should be able to figure that out.

And by the way – I'm [hpurmann on Keybase](https://keybase.io/hpurmann).
