---
author: steve@getrubix.com
date: Tue, 18 Mar 2025 14:28:34 +0000
description: '"Last time, we went over the basics of Intune Device Query and how to
  pull data from the Device category using KQL. If you’ve been playing around with
  it, hopefully, you’re starting to see just how powerful it can be. But what if you
  need data from multiple"'
slug: advanced-intune-device-query-joining-across-categories
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/advanced-intune-device-query-joining-across-categories_thumbnail.jpg
title: Advanced Intune Device Query Joining Across Categories
---

Last time, we went over the basics of **Intune Device Query** and how to pull data from the **Device** category using KQL. If you’ve been playing around with it, hopefully, you’re starting to see just how powerful it can be. But what if you need data from multiple categories?

Read the first part here: [https://www.getrubix.com/blog/getting-started-with-multi-device-query](https://www.getrubix.com/blog/getting-started-with-multi-device-query)

That's where the **join** comes in.

## Why You Need Joins
---

In the real world, your queries will often need data that lives in different categories. For example:

1. You might want **hardware details** (from a hardware category) along with **compliance status** (from the **Compliance** category).
    
2. Maybe you need to see all **autopilot-enrolled devices** and check their **TPM status** (from **Tpm**).
    
3. Or you could be troubleshooting a performance issue and need to combine **OS Version** with general device info from **Device**.
    

To do this, we need to **join** multiple datasets together.

## The Join Operator
---

In KQL, **join** lets you combine two datasets based on a shared value—like a **Device ID** or **Entra ID**. It works like an **inner join** in SQL, meaning it will only return rows that match in both datasets.

Here's the basic syntax:

```
Category 1
| join Category2
```

## Example: Finding Microsoft Devices with 8GB RAM
---

Let's say we want to see **Microsoft-manufactured devices** that have **exactly 8GB of RAM** and display their **Device Name, Device ID, and Entra Device ID**.

This is what our query would look like:

```
Device
| where Manufacturer == "Microsoft"
| join MemoryInfo
| where PhysicalMemoryTotalBytes == "8589934592"
| project DeviceName, DeviceId, EntraDeviceId
```

Let's break it down:

**Filter for Microsoft devices** - This limits the dataset to only devices made by **Microsoft** (e.g., Surface laptops and Hyper-V virtual machines).
```
Device
| where Manufacturer == "Microsoft"
```

**Join with MemoryInfo Category** - This brings in **MemoryInfo**, which contains RAM details for each device.

```
| join MemoryInfo
```

**Filter devices with exactly 8GB of RAM** - Since memory is measured in **bytes**, 8GB is **8 × 1024 × 1024 × 1024 = 8589934592 bytes**. This ensures we’re only looking at devices with **8GB of physical RAM**.

```
| where PhysicalMemoryTotalBytes = "8589934592"
```

**Select relevant columns**

```
| project DeviceName, DeviceId, EntraDeviceId
```

The final output displays:

>   **DeviceName**: Friendly name of the device
>
>   **DeviceId**: Unique identifier for the device in Intune
>   
>   **EntraDeviceId**: Corresponding ID in **Microsoft Entra ID**
    

Like anything else, the best way to get comfortable is to start writing your own queries and experiment. Let me know what you come up with!
