---
author: steve@getrubix.com
date: Fri, 13 Dec 2024 21:05:24 +0000
description: '"Wow- another week of Graph videos in the books.Check out the ever-growing
  playlist here.Today we’re going to spend some time talking about methods and how
  they play a critical part in working with the Microsoft Graph.What are methods?From
  the internet: REST API methods are the various ways that"'
slug: methods-of-madness
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/methods-of-madness_thumbnail.jpg
title: Methods of Madness
---

Wow- another week of Graph videos in the books.

Check out the ever-growing playlist [here](https://www.youtube.com/playlist?list=PLKROqDcmQsFls8cPHk3HFz2mUURHx46_O).

Today we’re going to spend some time talking about **methods** and how they play a critical part in working with the Microsoft Graph.

What are methods?
-----------------

> _From the internet: REST API methods are the various ways that clients can send requests to APIs. The API developer defines these methods when designing an API. Some REST API methods include_:

-   **GET**: Used to access resources from a server, such as reading data
    
-   **POST**: A method used in REST APIs
    
-   **PUT**: A method used in REST APIs
    
-   **DELETE**: A method used in REST APIs
    
-   **PATCH**: A method that applies partial modifications to a resource
    

In other words, they are actions. The URIs we constructed in [Part 1](https://www.getrubix.com/blog/microsoft-graph-a-beginners-guide-to-apis-endpoints-and-urls) are the endpoints that hold the data we want to work with, such as uses, devices, and apps. The **method** is _what_ we actually want to do with that data. Today we’ll take a look at **GET** and **POST**.

Getting stuff
-------------

The **GET** method is the simplest in terms of what it does and how it works. It just _gets_ stuff for us.

For example; when we want to retrieve all of our Intune managed PCs, we **GET** data from that call.  
`Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/beta/deviceManagement/managedDevices"`

It’s the same thing with users:  
`Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/beta/users"`

Posting and Creating
--------------------

Think of **POST** as the method to create. This is when you want to add something to a graph endpoint that didn’t exist before. When we use **GET** to make a request, we don’t need anything more than the endpoint itself. Let’s use groups for this example.

If I want to see all my groups, then my call is:  
`Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/beta/groups"`

But by adding the **POST** call to create a group, the call looks something like this:  
`Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/beta/groups" -Body <INSERT REQUEST BODY HERE>`

Whoa, whoa… what is this _body_ part of the call doing here? That is the information you will add to create your own group. Think about how you make a group today. You login to [https://entra.microsoft.com](https://entra.microsoft.com/) and navigate to **Groups** > **All groups** > **New group** and then enter in all the details.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1cd40063-4336-4b26-9cfc-7d82655577e4/newGroup.png)

Just because we’re doing this through the Graph doesn’t mean we can skip the details. We have to construct the body.

> Every endpoint has a different format for the body that gets posted, so be sure to check the [Microsoft Graph API documentation](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-1.0) to see examples of the body request.

Here is the data we need in JSON format for our request body to **POST** a group:

```
{
	"displayName" : "Name of your group",
	"description" : "Group description",
	"groupTypes" : [
		"Unified/Dynamic"
	],
	"mailEnabled" : true/false,
	"mailNickname" : "Your nickname here",
	"securityEnabled" : true/false
}


```

Some of these are fairly straight forward. Your group name and group description go at the top. Group types is an optional component. If you add it, the two choices are “Unified” and “Dynamic”. Unified will create a Microsoft 365 group. Dynamic will create…well… a dynamic group.

Mail and security enabled can be true or false depending on the group you want to create. However, even if this is not a mail enabled group, a nickname is still required for the request to post. With that in mind, let’s create a group.

If we use PowerShell for this, we can store the body in a script-block variable. Here is mine:

```
$body = @{
	displayName = "Blog writers group"
	description = "Group for my blog writers"
	securityEnabled = $true
	mailEnabled = $false
	mailNickname = "bloggroup"
} | ConvertTo-JSON


```

Notice I left out the “group types” because I don’t need it. If I were to just call that script block as a variable, this would be my output:

```
Name                           Value                                                                                                                                                               
----                           -----                                                                                                                                                               
mailEnabled                    False                                                                                                                                                               
description                    Another test                                                                                                                                                        
displayName                    GRAPH TEST GROUP 2                                                                                                                                                  
securityEnabled                True                                                                                                                                                                
mailNickname                   nickname      


```

Hmm. That looks nothing like the body we need. It has to be in JSON format. PowerShell has a cool cmdlet we can run called “ConvertTo-JSON”. This cmdlet takes our script block and conv…you know what? This one is pretty self explanatory.

By adding the ConvertTo-JSON cmdlet, now our body variable looks like this:

```
{
    "mailEnabled":  false,
    "description":  "Group for my blog writers",
    "displayName":  "Blog writers group",
    "securityEnabled":  true,
    "mailNickname":  "bloggroup"
}


```

There we go! Now I’ll just make the **POST** call and pass the body variable to the body parameter:  
`Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/beta/groups" -Body $body`

If all goes well, you should see a response back in PowerShell similar to this:

```
Name                           Value                                                                                                                                                               
----                           -----                                                                                                                                                               
writebackConfiguration         {isEnabled, onPremisesGroupType}                                                                                                                                    
securityEnabled                True                                                                                                                                                                
createdByAppId                 14d82eec-204b-4c2f-b7e8-296a70dab67e                                                                                                                                
groupTypes                     {}                                                                                                                                                                  
displayName                    Blog writers group                                                                                                                                                  
membershipRule                                                                                                                                                                                     
mail                                                                                                                                                                                               
theme                                                                                                                                                                                              
serviceProvisioningErrors      {}                                                                                                                                                                  
renewedDateTime                12/13/2024 8:42:58 PM                                                                                                                                               
onPremisesLastSyncDateTime                                                                                                                                                                         
preferredDataLocation                                                                                                                                                                              
description                    Group for my blog writers                                                                                                                                           
@odata.context                 https://graph.microsoft.com/beta/$metadata#groups/$entity                                                                                                           
isAssignableToRole                                                                                                                                                                                 
onPremisesProvisioningErrors   {}                                                                                                                                                                  
onPremisesNetBiosName                                                                                                                                                                              
resourceProvisioningOptions    {}                                                                                                                                                                  
preferredLanguage                                                                                                                                                                                  
onPremisesObjectIdentifier                                                                                                                                                                         
onPremisesSyncEnabled                                                                                                                                                                              
uniqueName                                                                                                                                                                                         
isManagementRestricted                                                                                                                                                                             
visibility                                                                                                                                                                                         
classification                                                                                                                                                                                     
mailNickname                   bloggroup                                                                                                                                                           
onPremisesSamAccountName                                                                                                                                                                           
resourceBehaviorOptions        {}                                                                                                                                                                  
infoCatalogs                   {}                                                                                                                                                                  
deletedDateTime                                                                                                                                                                                    
membershipRuleProcessingState                                                                                                                                                                      
proxyAddresses                 {}                                                                                                                                                                  
organizationId                 32f297ee-70e0-4dee-95b3-3146e5b2f929                                                                                                                                
mailEnabled                    False                                                                                                                                                               
securityIdentifier             S-1-12-1-76547248-1111012763-3758671015-579730705                                                                                                                   
createdDateTime                12/13/2024 8:42:58 PM                                                                                                                                               
onPremisesDomainName                                                                                                                                                                               
id                             049004b0-b59b-4238-a7c4-08e011fd8d22                                                                                                                                
expirationDateTime                                                                                                                                                                                 
onPremisesSecurityIdentifier
```

When I look in Entra, I can see my group is there as well.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a178205e-4381-42b2-ac9e-9093c50f51aa/postBlog2.png)

Next steps
----------

I would encourage everyone to take these steps and apply them to something you’re trying to create in your environment or lab. Depending on the endpoint, there are so many attributes you can customize- just look above!

Next time we’ll get into **PATCH** and **DELETE**. Thanks for reading, and we’ll be seeing you.
