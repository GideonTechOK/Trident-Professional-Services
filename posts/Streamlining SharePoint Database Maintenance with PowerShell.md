#Streamlining SharePoint Database Maintenance with PowerShell  

Managing SharePoint databases efficiently is crucial for maintaining optimal performance and ensuring smooth operations during patching and upgrades. In this blog post, we will discuss two essential PowerShell scripts—**Dismount_DB.ps1** and **Mount_DB.ps1**—that help automate the dismounting and mounting of SharePoint content databases. These scripts are particularly useful during maintenance windows when applying patches or updates to your SharePoint farm.

<!--more-->

---

## Understanding the Scripts

### Dismount_DB.ps1

The **Dismount_DB.ps1** script performs the following tasks:

- **Loads the SharePoint PowerShell Snap-In**: Ensures that the necessary SharePoint cmdlets are available.
- **Retrieves All Content Databases**: Uses `Get-SPContentDatabase` to gather all content databases in the farm.
- **Exports Database Information**: Saves details like Name, Data Source, and Web Application URL to a CSV file named `contentdbs.csv`.
- **Dismounts Each Content Database**: Iterates through each database and dismounts it using `Dismount-SPContentDatabase`, suppressing confirmation prompts.

#### Script Breakdown:

```powershell
# Add SharePoint PowerShell Snap-In 
if (!(Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue)) {
    Add-PSSnapin "Microsoft.SharePoint.PowerShell"
}

# Get all SharePoint Content Databases and Export to CSV
$contentDBs = Get-SPContentDatabase

if ($contentDBs) {
    $contentDBs | Select-Object Name, NormalizedDataSource, @{Name = "WebApp"; Expression = {$_.WebApplication.Url}} | Export-Csv "contentdbs.csv" -NoTypeInformation -Force
    $contentDBs | ForEach-Object {
        "$($_.Name),$($_.NormalizedDataSource)"
        Dismount-SPContentDatabase $_ -Confirm:$False
    }
}
```

---

### Mount_DB.ps1

The **Mount_DB.ps1** script handles the remounting of content databases after maintenance:

- **Loads the SharePoint PowerShell Snap-In**.
- **Imports Database Information**: Reads from the previously exported `contentdbs.csv`.
- **Defines Maximum Parallel Threads**: Sets the `$MaxThreads` variable to control concurrent operations.
- **Mounts Databases in Parallel**: Uses `Start-Job` to initiate mounting jobs, respecting the maximum threads limit.
- **Waits for Completion**: Monitors all jobs until they finish and then displays the results.

#### Script Breakdown:

```powershell
# Add SharePoint PowerShell Snap-In 
if (!(Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue)) {
    Add-PSSnapin "Microsoft.SharePoint.PowerShell"
}

# Import CSV file from current directory
$contentDBs = Import-Csv "contentdbs.csv"

# Maximum number of upgrades that will be run at a time
$MaxThreads = 4

# Define the script block for the upgrade process
$ScriptBlock = {
    param ($Name, $WebApp, $NormalizedDataSource)

    Add-PSSnapin "Microsoft.SharePoint.PowerShell"
    Mount-SPContentDatabase -WebApplication $WebApp -Name $Name -DatabaseServer $NormalizedDataSource
    Start-Sleep 50
}

# Execute the upgrades in parallel
foreach ($contentDB in $contentDBs) {
    while (@(Get-Job | Where { $_.State -eq "Running" }).Count -ge $MaxThreads) {
        Write-Host "Waiting for open thread...($MaxThreads Maximum)"
        Start-Sleep -Seconds 3
    }
    Start-Job -Name $contentDB.Name -ScriptBlock $ScriptBlock -ArgumentList $contentDB.Name, $contentDB.WebApp, $contentDB.NormalizedDataSource
}

# Wait for all jobs to complete
while (@(Get-Job | Where { $_.State -eq "Running" }).Count -ne 0) {
    Write-Host "Waiting for background jobs..."
    Get-Job
    Start-Sleep -Seconds 3
}

# Get results and remove the jobs
$Data = foreach ($Job in (Get-Job)) {
    Receive-Job $Job
    Remove-Job $Job
}

# Display the results
$Data | Format-Table -AutoSize
```

---

## How to Use These Scripts

### Before You Begin

- **Permissions**: Ensure you have administrative privileges on the SharePoint farm and the SQL Server instances.
- **Backup**: It's recommended to back up your content databases before performing dismount and mount operations.
- **Maintenance Window**: Perform these operations during a scheduled maintenance window to minimize impact on users.

### Dismounting Databases

1. **Run Dismount_DB.ps1**: Execute the script in PowerShell with administrative rights.
2. **CSV Export**: The script will generate `contentdbs.csv` containing details of all dismounted databases.
3. **Verification**: Confirm that all content databases have been successfully dismounted.

### Applying Patches or Maintenance

- Proceed with your patching or maintenance tasks on the SharePoint farm and databases.

### Mounting Databases

1. **Run Mount_DB.ps1**: Execute the script to remount the content databases.
2. **Parallel Processing**: The script will mount databases in parallel, as defined by `$MaxThreads`.
3. **Monitor Progress**: The script provides output on job statuses; wait until all jobs have completed.
4. **Verification**: Ensure all databases are mounted and accessible.

---

## Important Considerations

- **Impact on Users**: Dismounting databases will make associated sites unavailable. Inform users ahead of time.
- **Error Handling**: Review script outputs and logs to identify and address any errors.
- **Customization**: Adjust `$MaxThreads` based on your server capacity and performance considerations.

---

## Conclusion

Automating the dismounting and mounting of SharePoint content databases simplifies maintenance tasks and reduces downtime. By using these PowerShell scripts, administrators can efficiently manage databases during patching or upgrades, ensuring a smoother and more reliable operation of the SharePoint environment.

---

*Written by The Ideas Forge Team*

*Tags: #SharePoint #PowerShell #DatabaseMaintenance #Automation #Scripting*
