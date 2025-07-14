---
author: steve@getrubix.com
date: Tue, 24 Dec 2024 20:39:59 +0000
description: '"Hopefully, you''re not reading this. You should be getting ready for
  Christmas by decking halls, manufacturing gingerbread buildings, preparing for the
  coming of Mr. Clause- all that stuff. But you''re here. Which sadly means, you''re
  a bit like me (my family sends their condolences). Just because the holidays"'
slug: patch-and-delete-for-the-holidays
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/patch-and-delete-for-the-holidays_thumbnail.jpg
title: PATCH and DELETE for the Holidays
---

Hopefully, you're not reading this. You should be getting ready for Christmas by decking halls, manufacturing gingerbread buildings, preparing for the coming of Mr. Clause- all that stuff. But you're here. Which sadly means, you're a bit like me (my family sends their condolences).

Just because the holidays are here doesn't mean we just stop talking about graph stuff.

So today we're going to continue the **method** conversation and talk about **PATCH** and **DELETE**.

Remember: there is a full playlist about using the Microsoft Graph on our YouTube channel [here](https://www.youtube.com/playlist?list=PLKROqDcmQsFls8cPHk3HFz2mUURHx46_O).

PATCH: Making Targeted Updates
------------------------------

PATCH is perfect for making incremental changes to resources. Unlike PUT, which overwrites the entire resource, PATCH modifies only the specified fields. Here are some detailed examples to get you started:

### Example 1: Updating a Group's Display Name

Suppose you want to update a group’s display name. Here’s how:

```powershell
$patchBody = @{
    displayName = "Updated Group Name"
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method PATCH -Uri "https://graph.microsoft.com/v1.0/groups/{group-id}" -Body $patchBody
```

This command updates the `displayName` field while leaving other group attributes unchanged.

* * *

### Example 2: Updating Multiple Attributes of a Group

What if you need to update both the group description and its visibility? PATCH supports multiple changes in a single call:

```powershell
$patchBody = @{
    displayName = "Updated Group Name"
    description = "Updated group description"
    visibility = "Private"
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method PATCH -Uri "https://graph.microsoft.com/v1.0/groups/{group-id}" -Body $patchBody
```

* * *

### Example 3: Updating a User's Profile

PATCH can also be used for user objects. For instance, to update a user’s job title and department:

```powershell
$patchBody = @{
    jobTitle = "Senior Cloud Engineer"
    department = "IT Operations"
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method PATCH -Uri "https://graph.microsoft.com/v1.0/users/{user-id}" -Body $patchBody
```

This command modifies the `jobTitle` and `department` fields, leaving other profile details intact.

* * *

### Example 4: Nested Attributes and Complex Updates

For nested attributes like extension attributes, ensure proper JSON formatting:

```powershell
$patchBody = @{
    extensionAttributes = @{
        extensionAttribute1 = "Custom Value 1"
        extensionAttribute2 = "Custom Value 2"
    }
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method PATCH -Uri "https://graph.microsoft.com/v1.0/users/{user-id}" -Body $patchBody
```

Using the `-Depth` parameter ensures nested attributes are formatted correctly, enabling smooth updates.

JSON Formatting Best Practices
------------------------------

Proper JSON formatting is critical for PATCH requests. Here’s what to keep in mind:

1.  **Include Only Necessary Fields**: Only specify the fields you intend to update. Unnecessary fields increase the risk of errors.
2.  **Use `ConvertTo-Json` with `-Depth`**: For nested properties, use `-Depth` to ensure proper formatting.
    -   Example: `ConvertTo-Json -Depth 10` handles deeply nested objects.
3.  **Avoid Trailing Commas**: Ensure your JSON doesn’t have trailing commas, which can cause syntax errors.
4.  **Validate Your JSON**: Use tools like [JSONLint](https://jsonlint.com/) to validate your JSON structure before sending the request.
5.  **Escape Special Characters**: If your values include special characters (e.g., quotes), escape them properly.

Here’s an example of a properly formatted JSON body:

```json
{
    "displayName": "Updated Group Name",
    "description": "Updated group description",
    "visibility": "Private"
}
```

DELETE: Cleaning Up Resources
-----------------------------

DELETE is straightforward and removes resources permanently. Use it carefully, as there’s no recovery option.

### Example: Deleting a Group

To delete a group:

```powershell
Invoke-MgGraphRequest -Method DELETE -Uri "https://graph.microsoft.com/v1.0/groups/{group-id}"
```

Once executed, the group is removed from your environment. Always verify the resource ID before deletion to avoid mistakes.

Practical Tips
--------------

1.  **Test First**: Always test PATCH and DELETE commands in a lab environment before running them in production.
2.  **Log Actions**: Keep detailed logs of your API calls and responses for auditing and troubleshooting.
3.  **Double-Check IDs**: Especially for DELETE, ensure the correct resource ID is being targeted.

Conclusion
----------

PATCH and DELETE are essential tools for managing Microsoft Graph resources effectively. PATCH allows for precise, targeted updates, while DELETE helps remove unnecessary clutter. By following best practices and testing your calls, you can master these methods and streamline your workflows. Try them out and experience the difference!
