---
title: "VULTR: Instances sold out"
date: 2017-07-23T21:12:32
feature_image: https://images.unsplash.com/photo-1443159805125-50ae78d2d00d?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=9864fbc130fb3e5cce9803e540796d15
summary: 'While taking a closer look at VULTR I became very interested in their "$2.50 plan", as it is half as much as you would have to pay for the same performance at similar providers.'
tags:
  - hosting
  - tech
languages:
  - en
---

Now I wanted to deploy my permanent test and development VPS and was surprised to see that they were "sold out" in almost all regions. It astonished and confused me that a virtual instance could be sold out, even though I was able to choose every other, even bigger instance type.

I immediately had the feeling that they want to artificially limit the creation of these small instances for either of two reasons:

* They cannot cope with the high demand, i.e. need to add additional hardware or other resources to make sure to keep up with demand
* The $2.50 instances is more popular than expected and they cannot cover their mixed costs estimates

I was curious, so I reached out to their Support Team and quickly got a reply:

> Thank you for your interest in our $2.50 ‘sandbox’ plan. The plan is currently available in limited quantities in our Miami and New York/New Jersey locations. **Why is this awesome plan available in select locations?** It boils down to supply and demand. In order to maintain the high performance you’ve come to know and love at Vultr we limit the number of sandbox plans on each physical host node. This helps ensure the absolute best performance for our production-ready instances. **Will this inventory be replenished in other locations?**  
> Yes. We expect additional sandbox capacity will come online soon. Please visit <https://my.vultr.com/deploy> periodically and consider following us on Twitter for updates and other breaking news.

That answer did not feel very satisfying, so I contacted them again and wrote about my thoughts what the reason behind this limitation could be. I received the following additional message:

> On a purely hypothetical basis, I ask that you consider the following: We offer a 24-core instance as a large deployment for important workloads. A single instance of this tier will likely be a much more pleasant neighbor to reside with, in a shared resource environment, when compared to 24 or more 1CPU, sandbox-style deployments; each of which will have differing peak periods of use and CPU usage profiles in general. As such, we have made the decision to ensure smooth and continual high performance on our nodes via limiting the amount of sandbox plans that we can offer per hardware hypervisor host. I hope this helps to clarify our position.Unfortunately, there are no email alerts for sandbox availability; for the latest news and updates, we’d recommend that you follow our Twitter account.

I have to say I am still not very satisfied with that answer. There is no "Sandbox" tag that I could see in UI that distinguishes the smallest plan from all the others. Also would I really be "more satisfied" if a 24-core instances was running as my "neighbour"?

I am still not convinced…

Whatever the real circumstances are, I think it would be great if VULTR would explain the reason and their intentions behind this limitation. Maybe in a more detailed blog post.

But even though I was not fully satisfied with the response, their support team was quick, professional and very corteous! A very good experience!

_If you are interested in VULTR and want to try it for yourself, I would like to ask you to use[my referral code](http://www.vultr.com/?ref=7149844). Thank you very much!_
