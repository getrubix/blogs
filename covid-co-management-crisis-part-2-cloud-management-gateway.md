---
author: steve@getrubix.com
date: Tue, 01 Sep 2020 13:26:22 +0000
description: '"I’ll be completely honest- when I first wrote Covid Co-Management Crisis
  back in April, I didn’t rush to work on the promised follow up about the cloud management
  gateway. As we all were, I was hoping we’d be done with this whole mess before it
  was needed."'
slug: covid-co-management-crisis-part-2-cloud-management-gateway
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/covid-co-management-crisis-part-2-cloud-management-gateway_thumbnail.jpg
title: Covid Co-Management Crisis Part 2 Cloud Management Gateway
---

I’ll be completely honest- when I first wrote [Covid Co-Management Crisis](https://www.getrubix.com/blog/covid-co-management-crisis) back in April, I didn’t rush to work on the promised follow up about the cloud management gateway. As we all were, I was hoping we’d be done with this whole mess before it was needed. While that was not the case, IT departments are still out there working hard everyday to make sure users are getting up and running with Windows 10 PCs. I’m fortunate enough to have helped many companies move to modern management in order to better support this year’s forced upon remote work situation.

Anyways, back in April I suggested that everyone enable co-management with the Microsoft Endpoint Manager to start shifting workloads from on-premise SCCM to Intune. If you haven’t done so already, there’s no reason not to. It’s the best way to enable cloud management and keep your remote PCs up-to-date and compliant.

![Screen Shot 2020-09-01 at 8.50.26 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598964674873-QJ3B0K66LNLP26HLD9CZ/Screen+Shot+2020-09-01+at+8.50.26+AM.png)

But for some, shifting everything to Intune takes a lot of effort, and that’s not a luxury most can afford while struggling to stay on top of current fleet management amidst the Covid craziness. So, question: how do we get full control over our SCCM managed Windows 10 machines remotely? Answer: the cloud management gateway (CMG).

You can read all the official good stuff [here](https://docs.microsoft.com/en-us/mem/configmgr/core/clients/manage/cmg/plan-cloud-management-gateway).

![Screen Shot 2020-09-01 at 8.50.13 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598964746984-IKOHWNGPJCY083JH2PAT/Screen+Shot+2020-09-01+at+8.50.13+AM.png)

In a nutshell, the cloud management gateway is an app-proxy that allows you to use Azure to authenticate clients to your on-premise SCCM instance via the internet without VPN. So for all of the tasks that have not been shifted to Intune yet, you can continue to leverage SCCM. Take a look at the high level authentication flow:

![Screen Shot 2020-09-01 at 8.37.22 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598963864200-EMSKG7UUN6B02C7Q6R5Z/Screen+Shot+2020-09-01+at+8.37.22+AM.png)

Now usually, I would advise against the implementation of a CMG. Or at least, suggest some long, hard consideration before recommending an organization set it up. This is because Intune is capable of almost all of the tasks you’d use the CMG and SCCM for, but at no Azure consumption charge. Take a look:

![Screen Shot 2020-09-01 at 8.42.56 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598964189974-FKY4PNYFZRBJ8KK0IMNW/Screen+Shot+2020-09-01+at+8.42.56+AM.png)

Because the CMG is an Azure service, there is a cost associated with it. The good news is that all content you push _from_ your on-premise instance into Azure cloud is free. The bad news is that content pushed from the CMG to a client has a cost, in addition to the standard ‘heartbeat’ checkin of the client through the CMG.

_For more accurate pricing info, read this fun little site: https://azure.microsoft.com/en-us/pricing/details/bandwidth/_

But hang on- what’s wrong with a cost associated with a service that lets you continue to manage your PCs regardless of their location? Well, nothing if there wasn’t a free alternative. You see, the CMG might have offered more value when Intune was not as developed as it is today. But what are you going to use your CMG for? Pushing applications, updates, monitoring, endpoint policy? Intune will do all of that for you today. Check this out in comparison to the above:

![Screen Shot 2020-09-01 at 8.48.36 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598964527727-8AD2ZR5UJY0ACPPM0JP7/Screen+Shot+2020-09-01+at+8.48.36+AM.png)

So wait… should I set this up or not? If it sounds like I’m not being clear, you would be right. There is no clear, one size fits all answer. Obviously, I’m an advocate for going as modern as you can, and knowing how capable Intune is means it would be well worth shifting everything over.

But I’m also a realist- I get it. That shift can be difficult. Even when engaging an experienced partner consultant to help, moving from SCCM over to Intune can still be a daunting road ahead. And couple that with the wonderful situation that 2020 has put us in, I believe the CMG is the perfect ‘halfway house’ until we can take the time to make that jump.

There are some things about co-management that aren’t always obvious, so hopefully I was able to clear that up. Armed with the right info, now you can decide if it’s worth setting up a CMG in your organization.

And in case you decide to go ahead and do this, stay tuned for a [step-by-step guide](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up) in setting it up…
