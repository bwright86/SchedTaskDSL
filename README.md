# SchedTaskDSL
Scheduled Tasks written in a DSL for ease-of-use.

## Overview
This is a Powershell module that provides a DSL structure for creating Scheduled Tasks on a Windows server.

## Structure
The structure of the DSL is laid out much like the GUI for creating a task, here is an example

``` Powershell
ScheduledTask "SyncFolders" {
    User = John.Smith
    Description = "Syncronizes two folders w/ robocopy."
    Hidden = $false

    Security -RunOnlyWhenLoggedOn -RunAlways -StorePassword {
        Credential = Import-Clixml -Path C:\Cred.xml
    }

    Trigger -OnASchedule -Weekly -SyncTimezones {
        StartTime = [DateTime]"11/21/2018 18:00:00"
        Recurrence = 2 #Days/Weeks - Depending on use of -Daily or -Weekly
        WeekDays = "Monday", "Wednesday", "Friday"
    }

    Trigger -OnASchedule -Monthly -SyncTimezones {
        StartTime = [DateTime]"11/21/2018 18:00:00"
        Recurrence = 2 #Days/Weeks - Depending on use of -Daily or -Weekly
        Months = "January", "April", "July", "October"
        Weeks = "First", "Fourth", "Last"
        WeekDays = "Monday", "Wednesday", "Friday"
    }

    Action Powershell {
        ScriptPath = "C:\PSScripts\SyncFolder.ps1"
        Arguments = "-Source 'C:\Source' -Destination 'C:\Destination' -Mirror:$true"
    }
}
```

# Security Keyword
The switches for the Security keyword are:

| Value | Description
|---|---
| `RunWhenLoggedOn` | Run this task only when the user is logged into the machine.
| `RunAlways` | Run the task regardless if the user is logged in. (If the task is executing against remote resources, it will require a username and password to be stored, use `-StorePassword parameter, along with a credential object inside the scriptblock.)
| `StorePassword` | Stores the username/password to impersonate when the task executes. Allowing non-local resources to be used in the script (i.e. Make a call to a website/DB/Rest-API/etc..., perform PSRemoting execution, use WMI/CIM session on a remote system, *You get the idea!*)

# Trigger Keyword
The switches for the Trigger keyword are:

| Value | Description
| ---: | ---
| `OnASchedule` | Run the task at a scheduled interval. (Additional parameters include: `-OneTime` \| `-Daily` \| `-Weekly` \| `-Monthly` and \[ `-SyncTimezones` \] )
| `AtLogOn` | Run the task when a user logs into the system.
| `AtStartup` | Run the task when the system boots into Windows.
| `OnIdle` | Run the task during idle periods (TODO: Add more details to this trigger.)
| `OnEvent` | Run the task when a specific event is logged in the event log.
| `OnCreateEdit` | Run the task upon creation or modification of this task.
| `OnConnection` | Run the task when any user or a specific user connects locally or from a remote system. ( Additional parameters include \[ `-AnyUser` \| `-Username` \] and \[ `-RemoteConnection` \| `-LocalConnection` \])
| `OnDisconnect` | Run the task when any user or a specific user disconnects locally or from a remote system. ( Additional parameters include \[ `-AnyUser` \| `-Username` \] and \[ `-RemoteConnection` \| `-LocalConnection` \])
| `OnLock` | Run the task when any user or a specific user locks their desktop. ( Additional parameters include \[ `-AnyUser` \| `-Username` \])
| `-OnUnlock` | Run the task when any user or a specific user unlocks their desktop. ( Additional parameters include \[ `-AnyUser` \| `-Username` \])

The parameters included in the scriptblock include:

| Key | Values | Trigger (Switch) | Switch | Description
| --- | --- | --- | --- | ---
| StartTime | Any valid `[datetime]` | All Triggers | All switches | A time/date for the task to begin running.
| Recurrence | 1-999 | `-OnASchedule` | `-Daily` or `-Weekly` | Specifies the number of days or weeks for each recurrence. (TODO: Test upper bounds of this value.)
| WeekDays | Array of strings: Sunday-Saturday | `-OnASchedule` | `-Weekly` or `-Monthly` | An array of week days for the task to execute upon each recurrence. Example: Every 2 weeks, execute on Monday, Tuesday, and Wednesday.
| Months | Array of strings: January-December | `-OnASchedule` | `-Monthly` | An array of months for the task to execute.
| Weeks | Array of strings: First-Fourth and Last | `-OnASchedule` | `-Monthly` | An array of the week number for the task to execute.

# Action
This keyword performs an action for the task.

The switches for the Action keyword are:

| Value | Description
| --- | ---
| Powershell | Executes a Powershell script. Expects a .ps1 file that can require parameters as arguments.
| Cmd | Executes a .bat or .vbs script.