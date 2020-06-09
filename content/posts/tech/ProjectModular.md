---
title: "ProjectModular - Automate Tasks with PowerShell"
date: "2020-02-05"
tags: ["powershell","automation","blueteam"]
---

## What is ProjectModular?

ProjectModular is a set of scripts written in PowerShell that was created to automate basic sysadmin/support jobs. It was designed to be a super user friendly experience for those who do not have a lot of PowerShell experience. To do that, most variables call back to a central file that can be edited after download.

## Who can use ProjectModular?

ProjectModular has been written so that anyone (with or without PowerShell knowledge) could pick it up, change a few fields and use it in their organisation to begin automating some tasks.

## What can ProjectModular do?

There is a range of scripts in this project from very specific program scripts to generic scripts.
The core scripts are the account creation and termination scripts which work in Active Directory environments.
They both have add-ons that can be enabled in the config to add features like:
 - Automatic JIRA Helpdesk ticket creation/comments
 - Dynamics AX user creation
 - Spanning Office365 Backup applying/removing licenses
 - Email details to a manager/supervisor
 - Email meeting reminder to remove account
 - Self generating termination audit document
 - Download .pst from Office365

There are also individual scripts that are for general purpose like:
 - Clearing Chrome and Internet Explorer temporary files for all users on servers
 - Collecting all AD groups and users in those groups
 - Finding all services that are set to automatic but haven't started
 - Querying AD description fields to find what computer a user owns
 - Generating a random password
 - Sending an email from Outlook locally on machine
 - RoboCopy files and get emailed a report upon completion

Find more information [here](https://github.com/0ldMate/ProjectModular)
