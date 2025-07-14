---
author: steve@getrubix.com
categories:
- intune
- security
- powershell
- azure
date: Tue, 05 Nov 2024 03:50:29 +0000
description: "Alright—we made it this far. If you’re still with me, then hang on for this one because it’s not going to be pretty. Today is all about moving those Group Policy objects to Intune. Remember, if you’re enjoying this series, make sure to check out the video companion."
slug: getting-started-with-intune-part-4-dont-bring-your-garbage-to-the-cloud
tags:
- defender
- active directory
- security
- script
- powershell
- entra
- intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-4-dont-bring-your-garbage-to-the-cloud_thumbnail.jpg
title: Getting Started with Intune Part 4 Dont Bring Your Garbage to the Cloud
---

Alright- we made it this far. If you’re still with me, then hang on for this one because it’s not going to be pretty. Today is all about moving those Group Policy objects to Intune.

Remember, if you’re enjoying this series make sure to check out the video companion series on the [GetRubix YouTube member channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

Who made these in the first place?
----------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/376ccbe3-8339-4720-9da7-14686b3d3c50/gpo2.png)

Ugh

Let me guess; you didn’t create your organization’s GPOs. Some guy named Ted did, and Ted quit about three and a half years ago. Since then, you’ve been trying to make sense of his mess, but the idea of changing or creating a policy sends you reaching for the _Pepto Bismol_ (or is that just me?)

Trust me when I say that you’re not alone. Every organization has a guy named Ted who started building GPOs when Windows XP was hot, and no one bothered to check in on Ted to see if what he was doing made sense, let alone if he wrote anything down.

This makes the task of transitioning to Intune policy a bit more difficult because before you decide what you want to move over, you have to first understand what you have. Luckily, Microsoft threw us a bone here with the [Group Policy analytics tool](https://learn.microsoft.com/en-us/mem/intune/configuration/group-policy-analytics).

Analyze this
------------

This built-in tool allows Intune to read your GPOs and tell you what the equivalent policy is. The process is pretty simple:

1.  Export your GPOs in an .xml format
    
2.  Upload to Intune
    
3.  Wait for Intune to generate the report
    

Simple, right? Well, there is an issue.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b0f98107-ba03-4d54-9abf-c2f91482393b/gpo1.png)

It looks promising, but don’t be fooled

### What the analytics does (and what it doesn’t)

The Group Policy Analytics tool is helpful, but don’t expect it to do all the heavy lifting. Here’s a rough idea of what it can and can’t do:

-   **What It does**:
    
    -   Quickly show which policies you can transition to Intune
        
    -   Identify settings that Intune can’t handle, so you can start thinking about alternatives (or resigning yourself to giving them up).
        
    -   Provide recommendations on equivalent Intune settings for certain GPOs, where possible (sometimes).
        
-   **What it doesn’t**:
    
    -   Understand context; there are many times where it’ll stare at a policy and tell you to your face it is not supported, when we all know there is a perfectly good Intune setting that does the same thing.
        
    -   Explain why certain settings aren’t supported - like I said, it doesn’t understand _all_ the policies, and for the ones that really aren’t supported anymore, well, who knows why?
        
-   **What it does but shouldn’t**:
    
    -   MIGRATE SUPPORTED POLICY
        

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/23f0a3c1-de23-4d51-bba6-3cfb0a451e56/gpo3.png)

But, it’s so tempting…

“Wait a minute, Steve”, you might be saying right now… “why wouldn’t I want to migrate the policy?”

I’m glad you asked.

### Just because you can, doesn’t mean you should

Remember, these aren’t your policies. These are Ted’s. And Ted is gone. If Active Directory, domain join, and GPOs represent the last several decades of PC management, then Intune, Entra Join, and the cloud represent the next ones. Why start by bringing over all of Ted’s crap from Windows XP?

> Think about the smartphone (this has a point, just stay with me).
> 
> When smartphones started entering the enterprise, we needed a way to secure them. Enter MDM (Mobile Device Management). This was a completely new platform for managing these devices in our environment. But how did we know what policy to apply? It wasn’t like we could look at the previous settings… because they DIDN’T EXIST!
> 
> We needed to look at the tools that MDM provided to us, decide what we wanted smartphones to do and not do, and create policies to match the needs of the business and the users.
> 
> Do you see where I’m going?

I know these are still PCs, but we have to look at modern management through the lens of the modern business needs, not which control panel settings we hid in Windows 7.

Step 1: Focus on what’s important
---------------------------------

Before we get into transferring and creating policies, we need to understand what they should be doing. This will most likely involve talking to your security folks.

-   **Gather the parameters**:
    
    -   Get a clear directive of the security posture that your PCs need to meet. Don’t worry about which policies were set before. Business needs change along with Windows. What was important in 2001 is most likely irrelevant today, so get some fresh requirements.
        
-   **Stop caring about ridiculous things**:
    
    -   This one is going to hurt, but you need to hear it. Stop enforcing wallpaper. Stop customizing desktop shortcuts. Stop pinning apps to the start menu and taskbar. Just stop. Your users aren’t the same demographic as they were in the 90’s.
        
    -   Your users are customers. Customers who wait in lines for a new iPhone every year. Customers who use tablets and app stores every day. There was a time when your users couldn’t install a piece of software if their life depended on it. Now they can.
        
    -   Policies should focus on securing the PC and making it productive. Not dumbing down or restricting the experience.
        

Step 2: Take only what you need to survive
------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ef9c1907-a64f-42f8-8bf8-d1bde4ae4a0f/gpo4.jpg)

Great movie

Don’t just copy everything over to Intune. You have to make choices and make them based on what is important.

### What to Migrate (The Bare Minimum)

There are a few policies you can’t just ignore, like:

-   **Security settings**:
    
    -   If you were able to successfully interface with your security team, then you should know what’s important. This will most likely include things like:
        
        -   Disk encryption
            
        -   Firewall rules
            
        -   Local account protection
            
        -   Antivirus settings
            
-   **App configurations**:
    
    -   Any GPOs that manage core apps like _Chrome_, _Webex_, _Office 365_; those should probably stay. Migrate them into Intune.
        

### We have the TECHNOLOGY; we can rebuild them

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/38900f0b-2a76-494b-a66f-829dfd67a312/gpo5.png)

Now we’re just going off the rails, but I’m enjoying myself

Some GPOs are designed to work in a local network, so they won’t do much for users working from home or in a coffee shop. These will need to be rethought for a modern approach:

-   **On-premises access**:
    
    -   Things like printer mapping, network drive connections, and file shares—probably not going to work as they are. If your organization hasn’t embraced cloud printing and OneDrive, you’re going to need to look at alternative solutions for making these work.
        
    -   PowerShell comes in handy at a time like this. There are also a ton of community tools, including my own [GitHub repo](https://github.com/stevecapacity/IntunePowershell) filled with random Intune scripts.
        

### Throw it away!

The truth is many of Ted’s policies are trash. They were built for Windows XP and 7 (and maybe Vista). You don’t need them for Windows 11 in 2024. Toss them in the bin with the floppy drives.

-   **UI stuff**:
    
    -   Settings to make Windows 10 look like XP or suppress user interface customizations. As I already told you, stop it. These need to go, and go fast.
        
-   **Legacy security settings**:
    
    -   Anything that involves legacy protocols, LM hashes, and other domain-centric settings do not matter anymore. These PCs will be Entra ID joined, _A.K.A._, cloud-native.
        

Step 3: Start fresh
-------------------

Congratulations: you’ve successfully navigated through Ted’s policies and understand what you need to do next. Fortunately, Intune makes this part painless.

Well, mostly painless.

-   **Security baselines**:
    
    -   These come out-of-the-box with Intune and are available for Windows, Defender, Edge, and Cloud PCs.
        
    -   Think of baselines as the ‘greatest hits’ of device policy. They’re designed to be best practices, but make sure you go through them and tweak as needed.
        
-   **Settings catalogue**:
    
    -   Intune offers a complete list of all Windows 10 and 11 settings that can be individually assigned or configured as a group. If there’s something you need that isn’t in the baseline, you’ll find it here.
        
    -   The search mechanism is decent and out of all the things in Intune, the GUI is pretty easy to navigate.
        
-   **ADMX templates**:
    
    -   Do you really need to configure something that Intune doesn’t have a setting for? Well guess what: virtually all of the Windows administrative templates that made up group policy have been moved to Intune and can be set the exact same way… maybe I should have opened with that?
        
    -   You can also import 3rd party policy for apps like _Chrome_, _Firefox_, _Webex_, and anything else that that publishes its own policy.
        

No more training, do you require
--------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/eda04b3b-6319-40bf-b896-fb19f3d733f2/gpo6.gif)

Nice

What is up with me and the movie puns in this one? Who knows.

What I do know is that you can do this. I have helped over 1,500 organizations go through this exact process, and I started _before_ Intune had the tools to help. Run a Google search for MMAT and see what comes up.

The reality is that the transition from Group Policy to Intune settings is no longer a technical challenge, but a mental and emotional one. Ted put a lot of work into those policies. He built his whole career on mastering Active Directory, the domain, and the art of policy inheritance.

Maybe your Ted is still there; maybe he isn’t. Maybe you have a whole bunch of Teds that are set in their ways. But cloud native Windows management is no longer a cutting edge, nice-to-have, niche idea.

It is the norm. It is the Microsoft recommended path forward. And it also makes sense for today’s workforce. Why would you want to only manage devices when they’re on your network? It just doesn’t make sense anymore.

If you’re implementing Intune, Autopilot, and you’re embracing cloud-native management, then you have to Entra join them. And that means moving these darn GPOs!
