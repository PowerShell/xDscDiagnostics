[![Build status](https://ci.appveyor.com/api/projects/status/p2vv1iwqcks4sk8m/branch/master?svg=true)](https://ci.appveyor.com/project/PowerShell/xdscdiagnostics/branch/master)

# xDscDiagnostics

The **xDscDiagnostics** module contains two cmdlets for analyzing DSC event logs and identifying the causes of any failure in a DSC operation: **Get-xDscOperation** and **Trace-xDscOperation**.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Contributing
Please check out common DSC Resources [contributing guidelines](https://github.com/PowerShell/DscResource.Kit/blob/master/CONTRIBUTING.md).

## Description

The **xDscDiagnostics** module exposes two primary functions - **Get-xDscOperation** and **Trace-xDscOperation** - and one helper function - **Update-xDscEventLogStatus** - that aid in diagnosing DSC errors.
Here, we use the term DSC operation to indicate the execution of any DSC cmdlet from its start to its end.
For instance, Start-DscConfiguration and Test-DscConfiguration would form two separate DSC operations.
The cmdlets also let you diagnose operations run on other computers. More details about their usage is given below in the Details section.

## Install the module

### Install the stable version module.
* For PowerShell Version 5 and higher running the following steps:
    * `install-module xdscdiagnostics -Repository 'PSGallery'`
* For PowerShell Version 4 and prior.
    * Download the [zip of the master repo](https://github.com/PowerShell/xDscDiagnostics/archive/master.zip)
    * Unzip the zip file to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\xDscDiagnostics` making sure the root of the zip ends up in the root of that folder.

### Install the dev version module.
* For PowerShell Version 5 and higher running the following steps:
    * `Register-PSRepository -Name xDscDiagnosticsDev -SourceLocation https://ci.appveyor.com/nuget/xdscdiagnostics -InstallationPolicy Trusted -Verbose`
    * `install-module xdscdiagnostics -Repository 'xDscDiagnosticsDev'`
* For PowerShell Version 4 and prior.
    * Download the [zip of the dev repo](https://github.com/PowerShell/xDscDiagnostics/archive/dev.zip)
    * Unzip the zip file to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\xDscDiagnostics` making sure the root of the zip ends up in the root of that folder.

## Cmdlets

### Get-xDscOperation
This cmdlet lists statuses of the last few run DSC operations.
It returns an object that has information about the time that operation was created, whether the operation was successful or not, a handle to all the events generated by that operation, and the unique job identifier for that operation.
(Read [this blog](http://blogs.msdn.com/b/powershell/archive/2014/01/03/using-event-logs-to-diagnose-errors-in-desired-state-configuration.aspx) to understand the role of the job ID in DSC events.)
The cmdlet accepts the following parameters:
* **Newest**: Number of past operations you want to output.
By default, it will display details of the last 10 operations
* **ComputerName**: Name of the computer from which you'd like to collect the event diagnostic details.
The input can be an array of strings.
You would need to execute the command `New-NetFirewallRule -Name "Service RemoteAdmin" -Action Allow` on the remote computer in order to execute this operation on it.
* **Credential**: Credentials required to access the computer given in the ComputerName property

### Trace-xDscOperation 
Once we run Get-xDscOperation, we can see which of the operations were a failure/success.
Also, we can note a correlation between SequenceID and JobID for each operation.
Trace-xDscOperation takes either of these values as parameters and gives you a readable list of events generated by their respective DSC operation.
By default, Trace-xDscOperation will list all the events generated by the most recent DSC operation.
This cmdlet returns an object that contains properties such as event type, event message, and time of event creation.
The cmdlet accepts the following parameters: 
* **SequenceID**: This is a field present in the object returned from running Get-xDscOperation.
It identifies an operation run in the computer.
By specifying the sequence ID, all the events pertaining to the corresponding operation are returned.
* **JobID**: This is a GUID that is a prefix to all the events published by DSC, which uniquely identifies each operation.
It is also a field present in the object returned from running Get-xDscOperation cmdlet.
By specifying a JobID, this cmdlet will extract and display all events pertaining to the corresponding DSC operation.
* **ComputerName**: Name of the computer from which you'd like to collect the event diagnostic details.
The input can be an array of strings.
You would need to execute the command `New-NetFirewallRule -Name "Service RemoteAdmin" -Action Allow` on the remote computer(s) in order to execute this operations on it.
* **Credential**: Credentials required to access the computer given in the ComputerName property.

### Update-xDscEventLogStatus
This cmdlet helps us enable or disable any of the DSC event logs.
When the cmdlets Get-xDscOperation and Set-xDscOperation are used, they will output details from events generated in the enabled channels.
If the channel is disabled, a warning is issued on the PowerShell console.
By using the cmdlet Update-xDscEventLogStatus, you could enable the channel required to collect DSC events.
* **Channel**: This is a mandatory parameter that indicates which DSC channel status needs to be updated: { Analytic | Debug | Operational }
* **Status**: This is a mandatory parameter that indicates the final state of the channel: { Enabled | Disabled }
* **ComputerName**: Name of the computer on which you would like to set the log status.
You would need to execute the command `New-NetFirewallRule -Name "Service RemoteAdmin" -Action Allow` on the remote computer(s) in order to execute this operations on it.
* **Credential**: Credentials required to access the computer given in the ComputerName property.

### New-xDscDiagnosticsZip
This cmdlet generates a zip of DSC and DSC Extension diagnostics to send to support.
The output will be the name of the zip file.
The cmdlet will confirm by default.
* **Session**: This is an optional parameter of a PSSession to use to collect the diagnostics

### Get-xDscDiagnosticsZipDataPoint
This cmdlet returns the list a zip data point.  A data point has the following properties
* **Name**: A unique name for the data point.
* **Description**: A description of the data point.
* **Target**: The general area of diagnostics the datapoint targets.

## Versions

### Unreleased

* Fixed help formatting.

### 2.6.0.0

* Added JobId parameter set to Get-xDscConfiguration
* Added IIS binding collection

### 2.5.0.0
* Added ability for New-xDscDiagnosticsZip to only collect the `xDscDiagnosticsZipDataPoint` collection you specify by data point or by group (called target).
* Added Get-xDscDiagnosticsZipDataPoint
* Added ability for New-xDscDiagnosticsZip to collect IIS and HTTPErr logs

### 2.4.0.0
* Added collection of OData logs to New-xDscDiagnosticsZip
* Converted appveyor.yml to install Pester from PSGallery instead of from Chocolatey.

### 2.3.0.0

* Renamed Get-xDscDiagnosticsZip to New-xDscDiagnosticsZip CmdLet and aliased to Get-xDscDiagnosticsZip to prevent breaks
* Added the following datapoint to New-xDscDiagnosticsZip:
    * Collected local machine cert thumbprints
    * Collected installed DSC resource version and path information
    * Collected System event log
* Added more detailed tests for New-xDscDiagnosticsZip
* Added Unprotect-xDscConfigurtion to decrypt current, pending or previous mofs

### 2.2.0.0

* Add the Get-XDscConfigurationDetail cmdlet

### 2.1.0.0

* Add the Get-xDscDiagnosticsZip CmdLet 

### 2.0.0.0

* Release with bug fixes and the following cmdlets
    * Get-xDscOperation
    * Trace-xDscOperation
    * Update-xDscEventLogStatus

### 1.0.0.0

* Initial release with the following cmdlets 
    * Get-xDscOperation
    * Trace-xDscOperation


## Examples

### Display the status of last 20 DSC operations

This example will list the last 20 DSC operations to see if any of them failed.

```powershell
Get-xDscOperation -Newest 20
```

### Display the status of the last two DSC operations in computer XXYY after passing Credential $cred

This example lets you find the status of DSC operations run on another computer. *Note: this requires a credential.*

```powershell
Get-xDscOperation -ComputerName Temp-Computer.domain.com -Credential $cred -Newest 2
```

### Trace a DSC operation that has a specific job ID

This example displays all events generated by the DSC operation that was assigned a particular unique job ID.

```powershell
Trace-xDscOperation -JobId aa6b4f3e-53f9-4f02-a502-26028e7531ca
```

### Get events of the second to last operation run on the localhost machine

This example displays a list of events and their messages published by the DSC operation run second to last (i.e. the sequence ID assigned to it is 2).

```powershell
Trace-xDscOperation -SequenceId 2 -ComputerName localhost
```

### Get diagnostic events of operations run on multiple computers that use the same credential

This example displays the list of events and their messages from multiple computers, as long as the credential passed works for all of them.

```powershell
Get-xDscOperation -ComputerName localhost, tempcomputer.domain.com -Credential $cred
```

### Enable the DSC Analytic event log

This example shows how you can enable the DSC Analytic channel event log.
By default, this channel is disabled.
By using this cmdlet, you can enable the channel collect all DSC events using the other 2 xDscDiagnostics cmdlets.

```powershell
Update-xDscEventLogStatus -Channel Analytic -Status Enabled
```

### Gather diagnostics from the machine running DSC or DSC Extension
* [Install the Module](#install-the-module)
* Open an elevated PowerShell Windows
* Run:
```PowerShell
New-xDscDiagnosticsZip
```
* Email the Zip that pops up to your support contact

### Gather diagnostics for the DSC Pull Server and Windows data points.
* [Install the Module](#install-the-module)
* Open an elevated PowerShell Windows
* Run:
```PowerShell
Get-xDscDiagnosticsZip -DataPointTarget 'DSC Pull Server','Windows'
```
* Email the Zip that pops up to your support contact

### Gather diagnostics for only eventlog datapoints
* [Install the Module](#install-the-module)
* Open an elevated PowerShell Windows
* Run:
```PowerShell
Get-xDscDiagnosticsZip -includedDataPoint (@(Get-xDscDiagnosticsZipDataPoint).where{$_.name -like '*eventlog'})
```
* Email the Zip that pops up to your support contact


### Gather diagnostics from a PSSession to the machine running DSC or DSC Extension
* [Install the Module](#install-the-module)
* Open an PowerShell Windows
* Open the PSSession to the Azure VM as an administrator on the VM
* Run:
```PowerShell
New-xDscDiagnosticsZip -Session $SessionToVm
```
* Email the Zip that pops up to your support contact


### Get the details of the last operation DSC performed

* Gets the verbose details of the Configuration Status object you pass to it.

``` PowerShell
Start-DscConfiguration .\Example -Wait
Get-DscConfigurationStatus | Get-XDscConfigurationDetail
```

Example output

```
time                         type    message                                                                     
----                         ----    -------                                                                     
2016-03-16T12:45:17.756-7:00 verbose [tplunktower]: LCM:  [ Start  Set      ]                                    
2016-03-16T12:45:18.272-7:00 verbose [tplunktower]: LCM:  [ Start  Resource ]  [[Log]example]                    
2016-03-16T12:45:18.273-7:00 verbose [tplunktower]: LCM:  [ Start  Test     ]  [[Log]example]                    
2016-03-16T12:45:18.273-7:00 verbose [tplunktower]: LCM:  [ End    Test     ]  [[Log]example]  in 0.0000 seconds.
2016-03-16T12:45:18.273-7:00 verbose [tplunktower]: LCM:  [ Start  Set      ]  [[Log]example]                    
2016-03-16T12:45:18.274-7:00 verbose [tplunktower]:                            [[Log]example] example            
2016-03-16T12:45:18.274-7:00 verbose [tplunktower]: LCM:  [ End    Set      ]  [[Log]example]  in 0.0010 seconds.
2016-03-16T12:45:18.274-7:00 verbose [tplunktower]: LCM:  [ End    Resource ]  [[Log]example]                    
2016-03-16T12:45:18.278-7:00 verbose [tplunktower]: LCM:  [ End    Set      ]                                    
2016-03-16T12:45:18.279-7:00 verbose [tplunktower]: LCM:  [ End    Set      ]    in  0.5230 seconds.             

```


### Decrypt the current mof

* Decrypts the current mof LCM is using.  **Must be run as administrator**

``` PowerShell
Unprotect-xDscConfigurtion -Stage Previous
```

Example output

```
?/*
@TargetNode='localhost'
@GeneratedBy=tplunk
@GenerationDate=04/07/2016 16:54:16
@GenerationHost=localhost
*/

instance of MSFT_LogResource as $MSFT_LogResource1ref
{
SourceInfo = "::1::24::log";
 ModuleName = "PsDesiredStateConfiguration";
 ResourceID = "[Log]example";
 Message = "example";

ModuleVersion = "1.0";
 ConfigurationName = "example";
};
instance of OMI_ConfigurationDocument

                    {
 Version="2.0.0";

                        MinimumCompatibleVersion = "1.0.0";

                        CompatibleVersionAdditionalProperties= {"Omi_BaseResource:ConfigurationName"};

                        Author="tplunk";

                        GenerationDate="04/07/2016 16:54:16";

                        GenerationHost="localhost";

                        Name="example";

                    };
```
