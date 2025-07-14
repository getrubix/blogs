---
title: "PowerShell Best Practices for Intune Logging"
slug: powershell-best-practices-for-intune-logging
date: "Mon, 31 Mar 2025 14:43:28 +0000"
author: steve@getrubix.com
description: "Whether you like it or not, the ability to write PowerShell scripts is critical for effectively managing Windows devices with Microsoft Intune. I know a lot of folks have the sentiment of 'wait, so now I'm a developer? Thanks a lot, Microsoft!' But the way I see it..."
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/powershell-best-practices-for-intune-logging_thumbnail.jpg
---

Whether you like it or not, the ability to write PowerShell scripts is critical for effectively managing Windows device with Microsoft Intune. I know a lot of folks have the sentiment of "wait, so now I'm a developer? Thanks a lot, Microsoft!"

But the way I see it, PowerShell is more of a Windows tool than anything else, so you would need these skills regardless of your endpoint management platform, right? Either way, we need to be familiar with it, and specifically for Intune, there are some fundamental things that I know help me in day-to-day management that will absolutely help you too.

Today we will focus on logging.

### Watch the YouTube video here:

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/G0hEgD3ysQY?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

## PowerShell logging: the short, short, short version
---

We’re going to assume you’re already somewhat proficient at reading and writing PowerShell. This series is all about best practices. For learning PowerShell properly, there are about a billion and one places on the internet for that.

What I usually see for logging is some form of `Write-Host` or `Write-Output` to convey a message. For example:

```pwsh
$status = $true
Write-Host "The status is $status"
```

That will display the status in the console itself. If you want to store everything in a log file, you’re better off with `Write-Output` (since it’s going to somewhere else besides the host). To capture all of your output to a log file, you would use the `Start-Transcript` command.

```pwsh
Start-Transcript -Path C:\yourDesiredPath\nameOfYourLogFile.log
$status = $true
Write-Output "The status is $status"
Write-Output "This is the end of the script"
Stop-Transcript
```

This will result in a log file that looks like this:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1e584c6e-63ac-4eca-bd55-76c443d78d99/log1.png)

## Log Function
---

That’s fine to start, but we can do better. Let’s start by creating a log function as opposed to just typing `Write-Output` every time. With a function, we can call it with something shorter to type and provide a more robust output, like adding the date and time-stamp for each logged event. Something like this:

`2025-03-17 01:36:42 PM - The status is true`

Looks way cleaner. So here is the function.

```pwsh
function log{
	param(
		[string]$message
	)
	$time = Get-Date -Format "yyyy-MM-dd HH:mm:ss tt"
	$output = "$time - $message"
	Write-Output $output
}
```

It’s a fairly simple function. Our parameter is a message, so anything we would typically write after the `Write-Output` cmdlet. Then we’re just automatically adding the date/time prefix to it. Here is the same script but with the log function:

```
function log{
	param(
		[string]$message
	)
	$time = Get-Date -Format "yyyy-MM-dd HH:mm:ss tt"
	$output = $time - $message
	Write-Output $output
}

Start-Transcript -Path "C:\Path\LogFile.log"
$status = $true
log "The status is $status"
log "This is the end of the script"
Stop-Transcript
```

Here is what the result looks like:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d0c75683-8363-47c3-80c2-39d2305cac5b/log2.png)

## Where to put the logs?
---

You can write logs to just about anywhere on the PC. However, allow me to bring your attention back to Intune and the _“Collect Diagnostics”_ feature:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/deabf6d0-2f4a-4e45-a700-a408783aa2bd/log3.png)

Clicking this will pull all of Intune related events up to us in a neat, little file that we can collect remotely. Wouldn’t it be great if we could do the same with our own logs? Well, we can.

When you collect diagnostic data from Intune, it comes from this directory:

`C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`

Therefore, anything we write to that directory will come along with the rest.

All you have to do is specify when you start the transcript:

`Start-Transcript -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\myLogFile.log"`

That’s it!

## Next up, error handling
---

The cool thing about this logging method is that you can apply it to virtually any PowerShell scenario and get the same results (which is why I consider it a fundamental skill).

In the next part in the series, we’ll start talking about error handling because let’s face it- sometimes things just don’t work.
