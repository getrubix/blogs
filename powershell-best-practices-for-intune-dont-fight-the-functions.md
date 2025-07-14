---
author: steve@getrubix.com
categories:
- intune
- security
- powershell
date: Fri, 11 Apr 2025 15:38:06 +0000
description: "PowerShell functions provide a structured way to organize reusable code. In the context of Microsoft Intune and system administration, they enhance script readability, reduce duplication, and improve long-term maintainability. And while I completely understand not wanting to take “structured” advice from me, it will definitely help you."
slug: powershell-best-practices-for-intune-dont-fight-the-functions
tags:
- intune
- powershell
- compliance
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/powershell-best-practices-for-intune-dont-fight-the-functions_thumbnail.jpg
title: PowerShell Best Practices for Intune Dont fight the functions
---

PowerShell functions provide a structured way to organize reusable code. In the context of Microsoft Intune and system administration, they enhance script readability, reduce duplication, and improve long-term maintainability. And while I completely understand not wanting to take “structured” advice from me, it will definitely help you in the long run.

This guide explains what functions are and why you need to start incorporating them in your PowerShell Intune journey.

_\*There isn’t a corresponding video for this piece yet, that will come out Tuesday the 15th._

## What are PowerShell Functions?
---

A PowerShell function is a named block of code that performs a defined task. You declare it once and call it wherever needed in your script:
```pwsh
function Get-IntuneWelcome {
    Write-Output "Welcome to Intune. Please update your policies."
}

Get-IntuneWelcome
```

Functions can also take parameters, allowing for flexible and reusable logic:
```pwsh
function Get-IntuneWelcome {
    param(\[string\]$UserName)
    Write-Output "Hello $UserName, please ensure your device is compliant."
}

Get-IntuneWelcome -UserName "Steve"
```

## Why Use Functions Instead of Inline Code?
---

There are several reasons to define functions rather than writing inline code repeatedly:

-   **Avoid Duplication**: Reusing logic without copy-pasting improves maintainability.
    
-   **Improved Readability**: Named functions clarify the intent of each code block.
    
-   **Testing**: Functions can be tested independently.
    
-   **Error Isolation**: Errors can be managed more effectively when logic is encapsulated.
    

### Example Comparison: Inline Code vs. Function

This block of code manually checks the compliance status of two users, “Alice” and “Bob”, using Microsoft Intune cmdlets.

**Without a Function (Inline Repetition):**
```pwsh
$user = "Alice"
Write-Output "Checking compliance for $user"
$status = Get-IntuneManagedDevice -Filter "userPrincipalName eq '$user'"
Write-Output "Compliance Status: $($status.ComplianceState)"

$user = "Bob"
Write-Output "Checking compliance for $user"
$status = Get-IntuneManagedDevice -Filter "userPrincipalName eq '$user'"
Write-Output "Compliance Status: $($status.ComplianceState)"
```
Now here is the same code providing the same result but structured in a function:
```pwsh
function Check-UserCompliance {
    param(\[string\]$UserName)
    Write-Output "Checking compliance for $UserName"
    $status = Get-IntuneManagedDevice -Filter "userPrincipalName eq '$UserName'"
    Write-Output "Compliance Status: $($status.ComplianceState)"
}

Check-UserCompliance -UserName "Alice"
Check-UserCompliance -UserName "Bob"
```
The function version is more concise, avoids repetition, and improves readability.

## Characteristics of Well-Written Functions
---

While a function can literally be made up of whatever you want it to be, there are some general guidelines and best practices that will help structure your functions to be more useful.

An effective function in PowerShell should:

-   Use a descriptive name (e.g., `Get-ComplianceStatus`)
    
-   Accept parameters where appropriate
    
-   Return output in a consistent format
    
-   Include basic error handling
    

### **Example:**
```pwsh
function Get-DeviceCompliance {
    param(\[string\]$DeviceID)
    try {
        $status = Get-IntuneManagedDevice -DeviceId $DeviceID
        return $status.ComplianceState
    } catch {
        Write-Warning "Device not found: $DeviceID"
    }
}
```

## Start Small and User Your Imagination
---

The best way to get started with functions, is just like getting started with anything else; start doing it!

Do you find yourself copying and pasting the same lines of code over and over in a script? Make a function.

Are you running the same lines of code multiple times a day to achieve the same result? Put that thing in a function.

Start slow and try to solve problems that will make your life easier. In the future, we’ll go deeper into functions by introducing switches, custom output, pipelines, and more.
