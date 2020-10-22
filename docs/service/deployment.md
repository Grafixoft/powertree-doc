---
id: deployment
title: Deployment
---

All PowerTree binaries do not have any machine-specific installation and are designed to be runnable from the moment you unpack and apply the appropriate [configuration](configuration).

That being said, the executables themselves need to be delivered and kept running on the compute nodes.

In terms of infrastructure, the least you would need are one or more compute nodes, which can be any PC, virtual or otherwise, running inside the same network TODO: file share. You will also need a SQL Server 2016+ instance for the Operation Store database which should be accessible from within the nodes.

Logs from across the nodes are shipped to an existing Elasticsearch instance and later visualized in Kibana.

### Operation Store Deployment

The first step of setting up a PowerTree cluster is the SQL Server Operation Store deployment. 

In this example we'll explore how a developer can setup a LocalDB instance of the Operation Store, but with a few parameter tweaks it can be used to deploy a remote Operation Store as well.

This is easily done through an in-memory workflow:

1. Follow the steps outlined in the [Command-Line Interface](../service/cli) in order to start a PowerTree session in PowerShell
2. Replace the workflow parameters from the snippet below with your desired values
```powershell
$DatabaseName = "OperationStoreLocal"
$WorkflowDirectory = "{Path-To-OperationStore.SqlServer}"

Invoke-Workflow `
    -Assembly "OperationStore.SqlServer"`
    -Namespace "Grafixoft.Powertree.OperationStore.SqlServer"`
    -Class "DeployOperationStoreSqlServerWorkflow"`
    -FullDirectoryPath $WorkflowDirectory `
    -Parameters @{`
        ConnectionString="Data Source=(LocalDB)\MSSQLLocalDB; Integrated Security=SSPI; Initial Catalog=$DatabaseName";`
        DatabaseInitialSize="10MB";`
        DatabaseMaxSize="10000MB";`
        DatabaseFilePath="$env:TEMP\$DatabaseName.mdf";`
        DatabaseFileGrowth="10MB";`
        LogInitialSize="10MB";`
        LogMaxSize="10000MB";`
        LogFilePath="$env:TEMP\$DatabaseName.ldf";`
        LogFileGrowth="10MB";`
    }`
    -TimeoutSec 60
```

3. Execute the command
4. You should see the following in the PowerShell window:

```
   Status
   ------
Succeeded
```

### Service Fabric Deployment

Workflow hosts can be easily deployed to an existing Service Fabric cluster, whether on-premise or in the cloud. This greatly simplifies the process of managing compute node instances and upgrading service versions.

Again, this can be accomplished by in-memory execution of a workflow.

The deployment workflow uses X.509 certificate authentication and assumes the appropriate Service Fabric certificate has been installed in the specified Certificate Store for the current user.

If `$ExecutionEnvironmentPath` is provided, it is copied in package output folder.

An application package must be uploaded to an image store before it can be registered and deployed. For local clusters, the image store is a local file folder, e.g. `file:C:\SfDevCluster\Data\ImageStoreShare`. For Azure-hosted clusters, the image store is a hosted image-store service by default - `fabric:ImageStore`.

After the PowerTree package is verified, the corresponding application type is registered within the system.
Depending on whether there's an existing instance of the PowerTree application or not, the deployment process chooses to perform a fresh install or upgrade the cluster. The upgrade process of PowerTree in the cluster is monitored and executes health checks on every node. If health is in an error or warning state the whole upgrade is rolled back.

The example below assumes a cloud deployment
All that's needed is to adjust the parameters as nesessary and run `Invoke-DeployToServiceFabric`.

```powershell
function Invoke-DeployToServiceFabric {
    param(
        # Connection string to a primary node in the cluster, e.g. 172.168.0.52:19000
        [Parameter(Mandatory=$true)]
        [string] $ClusterConnection,
        # Path to a folder where the runtime binaries are located
        [Parameter(Mandatory=$true)]
        [string] $SourcePath,
        # Path to the Workflow.ServiceFabricDeployment DLLs
        [Parameter(Mandatory=$true)]
        [string] $WorkflowPath,
        # The Service Fabric certificate thumbprint
        [Parameter(Mandatory=$true)]
        [string] $CertificateThumbprint,
        # The common name of the Service Fabric certificate
        [Parameter(Mandatory=$true)]
        [string] $CertificateCommonName,
        # The certificate store name
        [Parameter(Mandatory=$true)]
        [string] $CertificateStoreName,
        # Temp path to store the package before uploading to Service Fabric
        [string] $PackageRootPath = "$env:TEMP\PowerTreePackage",
        [string] $ExecutionEnvironmentPath = $null,
        [string] $ApplicationName = "PowerTreeApp",
        [string] $Version = "1.0.0",
        [string] $ServiceName = "WorkflowHost"
    )

    $paramsList = @{`
        ClusterConnection = $ClusterConnection;`
        ImageStoreConnectionString = "fabric:ImageStore";`
        InstanceCount = -1;`
        ApplicationName = $ApplicationName;`
        Version = $Version;`
        ServiceName = $ServiceName;`
        EntryPoint = "Workflow.HostManager.exe";`
        IsSecuredCluster = $true;`
        CompressPackage = $false;`
        PackageRootPath = $PackageRootPath;`
        SourcePath = $SourcePath;`
        ExecutionEnvironmentSourcePath = $ExecutionEnvironmentPath;`
        CertificateStoreName = $CertificateStoreName;`
        CertificateThumbprint = $CertificateThumbprint;`
        CertificateCommonName = $CertificateCommonName;`
    }
    
    Invoke-Workflow `
        -Assembly "Workflow.ServiceFabricDeployment"`
        -Namespace "Grafixoft.Powertree.Workflow.ServiceFabricDeployment"`
        -Class "RuntimeDeploymentWorkflow"`
        -FullDirectoryPath "$WorkflowPath"`
        -Parameters  $paramsList `
        -TimeoutSec 600
}
```

### Manual Deployment

#### Compute Node Deployment

Bare metal deployment is the simplest deployment option. 
It consists of copying the PowerTree binaries and their respective configuration to all target compute nodes and setting up a daemon to keep the `Workflow.HostManager` process alive.



