---
author: steve@getrubix.com
date: Mon, 02 Oct 2023 14:44:26 +0000
description: '"I’ll be honest- when putting together the Intune tenant to tenant device
  migration solution, the user data was the last thing on my mind. It’s not because
  I think people should adopt OneDrive, be responsible for where they put their data,
  or just give up on their"'
slug: two-paths-for-user-data-migration
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/two-paths-for-user-data-migration_thumbnail.jpg
title: Two Paths for User Data Migration
---

I’ll be honest- when putting together the Intune tenant to tenant device migration solution, the user data was the last thing on my mind. It’s not because I think people should adopt OneDrive, be responsible for where they put their data, or just give up on their cat pics altogether. I was just focused on bigger things- like moving a PC between Intune tenants without wiping the device.

Well, once that part became fairly stable, it was time to address the elephant in the room that is user data.

### Watch the YouTube video here:

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/mUokZ2XB4Jg?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

## First pass
---

If you’ve been following my series on the solution, or even trying it out for yourself, then you’re aware we do address the user data. This is limited however. Here is what we have at the moment:

-   Organizations can set which data is migrated from a user’s profile _(AppData, Documents, Downloads, etc.)_
    
-   Data is stored to a temp location, later copied to new user profile, and then removed. This requires 3x the size of the user data to be available on a local machine. This is not ideal.
    
-   The solution only accounts for in an “in-place” tenant migration, meaning the user and existing PC are moved.
    
-   We’re not accounting for a user moving tenants and receiving a brand new PC.
    

## Options
---

Having options is a good thing, so we went back and looked at user migration from the perspective of two different scenarios: an **in-place** migration which occurs on the same PC, and a **hardware refresh** migration, which is when the user moves tenants and is issued a brand new PC. Let’s look at each solution.

### In-place Migration

This is similar to what we’ve already covered, but we’re now using Azure blob storage as an alternative to copying data on the local machine. Here is the high-level workflow:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/58bf1b53-fca5-4d59-8b19-c1434b9b5865/Screenshot+2023-10-02+at+8.40.16+AM.png)

Almost looks like a map to a theme park, right? Anyway, here’s the flow:

1.  Pre-migration backup PowerShell script is deployed before the tenant migration. This can be anytime from 5 days to a few weeks before the move, but I wouldn’t recommend waiting too long, as user data can change a lot in a few days.
    
2.  The script looks at the total user data size and compares to available disk space.
    
3.  If the PC has more than 3x the user data size in free space, we go the local route.
    
4.  If there isn’t that much space free, we check to see if 2x is available. If so, we compress the user data and upload it to Azure blob storage. Credentials are provided within the script.
    
5.  A variable is set as a flag to determine migrate method. So for post-migration, if the variable is set to _local_, the post-migration script runs the restore from the temp location, as detailed in our original solution.
    
6.  If the variable is set to _blob_, we then connect to the blob storage location, download the compressed files, and copy them to the new user directory.
    
7.  There is a third option. If there was not enough space to compress data in pre-migration, we set the variable as _NA_, and in that case do not migrate data. Organizations are free to explore alternate methods (more on that later).
    

### Hardware Refresh


This second method came about for two organizations I was working with while making the first series. Here, the user is migrated to a new tenant, but instead of bringing their existing PC, they are issued a new machine deployed via Autopilot. Take a look:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4943f015-6223-456a-8df0-d82859d236ce/Screenshot+2023-10-02+at+9.52.08+AM.png)

In this model, we have two options. We can use blob storage, but only as a fail safe. We’re using the pre-migration script to create an SMB share that can be read by our new Autopilot device, allowing for peer-to-peer network sharing. Here’s how it works:

1. Pre-migration PowerShell script runs on device in Tenant A.
    
2.  SMB share is created, along with a ‘reader’ account that has permission to the share. That info is stored in an XML file to be given to the new PC later.
    
3.  If 2x the size of user data is available in free disk space, the user data is compressed and added to blob storage.
    
4.  When the new Autopilot PC is deployed, the post-migration PowerShell script runs from Tenant B.
    
5.  The new PC attempts to connect to the SMB share on the original PC. If it can, the data us copied to the new user profile.
    
6.  If the PC cannot connect to the SMB share, it looks if data is stored in Azure, and can download the blob files. They are then restored to the user profile.
    

## Perfect, right?
---

Note that this is what has been effective for the organizations I work with. There are still some things that aren’t perfect, but with this newer method we have options. Over the next few weeks, I’ll be detailing every step in the process as well as updating our GitHub repo.

And of course, here is a video going over the new process.
