---
title: Fediverse Link Bot
date: 2018-08-26T23:05:50
feature_image: https://images.unsplash.com/photo-1527168027773-0cc890c4f42e?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=b8967a215ee951c7f2b2eb367042858f
summary: 'In the past it was easy to share interesting, or funny finds on the Internet. It was this magical place which was called “cyberspace” by the uninitiated.'
tags:
  - automation
  - socialmedia
  - tech
languages:
  - en
---

Only people who were really interested in this new medium were able to use it properly, as it was not that easy to get “online” at the time. The only way to reach people was plain text emails. No social media, no messengers, maybe an IRC channel if you were lucky.

# The era of the messengers

The more people “got online”, the more the communication got fragmented and compartmentalised. It started with the messengers in the 90s. First [ICQ](https://en.wikipedia.org/wiki/ICQ), [AIM](https://en.wikipedia.org/wiki/AIM_\(software\)), then [MSN](https://en.wikipedia.org/wiki/Windows_Live_Messenger) and [Yahoo](https://en.wikipedia.org/wiki/Yahoo!_Messenger). For a long time, whenever I dialled in via modem or ISDN card (or later, with a 768 kilobit DSL connection), I had to use three different messenger services to reach people.

# Digital communication nowadays

Nowadays the landscape is even more fragmented. There is [SMS](https://en.wikipedia.org/wiki/SMS), [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service), [WhatsApp](https://en.wikipedia.org/wiki/WhatsApp), [Telegram](https://en.wikipedia.org/wiki/Telegram_\(service\)), [Signal](https://en.wikipedia.org/wiki/Signal_\(software\)), [Facebook Messenger](https://en.wikipedia.org/wiki/Facebook_Messenger), [Facebook](https://en.wikipedia.org/wiki/Facebook), [Twitter](https://en.wikipedia.org/wiki/Twitter), [Discord](https://en.wikipedia.org/wiki/Discord_\(software\)), email… you name it! It got a lot harder to reach people, and it has become a lot of work to send an interesting story or a funny link to the right “audience”. And sometimes it even happens that you gravely annoy people with your links, maybe even to an extent where they mute you or unfollow you. Most of the time they just suffer quietly and politely, with the only hope that your “link storm” ends sooner rather than later.

# A different approach

I thought about this predicament for quite some time and was not sure what to do. After a while it dawned on me that I might need to change the approach that I used to share content. In 2018, people are used to be able to choose. Most of the intelligent people do not want to get force fed something, they want to be able to choose something, based on topics of interest.

Then I decided to built a “link portal”. A kind of “micro blog”, but for links. With a little title, maybe an excerpt or a few comments, and a lot of tags so people could pick what they wanted. I am no programmer, but I took it as an opportunity to learn Python and Django and started to code.

Shortly after I started – and after I saw that I totally underestimated the scope of this project – [Mastodon](https://en.wikipedia.org/wiki/Mastodon_\(software\)) became the topic of conversation again. People were fed up once more with Twitter and threatened to leave in tropes[^1] because of some [changes to their public API](https://9to5mac.com/2018/08/07/twitter-api-change-tweetbot-twitterrific/), which would result in rendering 3rd party clients useless. Of course I tested it, and it turns out, it was exactly what I was looking for.

# Make use of open standards

Mastodon uses [ActivityPub](https://en.wikipedia.org/wiki/ActivityPub) to create the [Fediverse](https://en.wikipedia.org/wiki/Fediverse). You can imagine it as Twitter on a community level, i.e. every community has its own Twitter, but you can also follow and communicate between these communities[^2]. So I created a new single user [Mastodon instance](https://click.ba.it/@links), which now can be followed by any Mastodon user in the Fediverse. It is, of course, not only limited to Mastodon users. You can use the web browser and look through the posts, or subscribe to new entries via [RSS](https://en.wikipedia.org/wiki/RSS) or [Atom](https://en.wikipedia.org/wiki/Atom_\(Web_standard\)).

# Embedded into a workflow

I want to share my findings, if possible, without much effort. If it becomes too much of an hassle to do something, you do it less or stop doing it altogether (blogging, exercising, anything really, it is human nature I guess). So I needed a workflow that was not unintrusive and not noticeable. And I think I found the perfect workflow for me.

I am using [Pocket](https://en.wikipedia.org/wiki/Pocket_\(service\)) for quite a while now to _categorise_ , _tag_ and _archive_ everything that I find intriguing, funny, fascinating or simply interesting. It works pretty well for me, as it is available on all platforms that I use and has a Developer API for integration. Combined with [IFTTT](https://en.wikipedia.org/wiki/IFTTT), it is very easy to “collect” web sites and links, but also effortless to send them to the bot for publication.

For example, I have an IFTTT service trigger for Twitter. Every first link of a tweet I _favourite_ is automatically sent to my Pocket list. The Pocket extension for Firefox and Chrome allows me to send any page I want to my Pocket reading list with just one click. Pocket’s user interface lets me tag my links as I see fit, and then, with just another click, I can _archive_ a link. Everything that is archived, is picked up by IFTTT and sent to [Marvin](https://en.wikipedia.org/wiki/Marvin_the_Paranoid_Android), my Mastodon bot.

Easy, unintrusive and uncomplicated!

{{< img src="https://media.jason.re/18/8/ifttt-mastodon-bot.png" >}}

[^1]: Every couple of months someone, or some groups, threaten to leave Twitter. Either because they cant use a 3rd party client no more, or because of the toxicity of the social network, or because someone was [wrong](https://xkcd.com/386/) on the Internet. They never do, or come back a short while later because of the [lock-in effect](https://jimsmarketingblog.com/2013/02/26/how-to-use-the-lock-in-effect-to-retain-your-clients-or-customers/). Roughly 99% of all threats/complaints I cannot take seriously anymore.

[^2]: It is a very interesting concept, and we will have to see if it goes the way of [Diaspora](https://en.wikipedia.org/wiki/Diaspora_\(software\)), or if it will develop into a strong long-term alternative to Twitter.
