---
id: deployment
title: Deployment
---

All PowerTree binaries do not have any machine-specific installation and are designed to be runnable from the moment you unpack and apply the appropriate [configuration](configuration).

That being said, the executables themselves need to be delivered and kept running on the compute nodes.

In terms of infrastructure, the least you would need are one or more compute nodes, which can be any PC, virtual or otherwise, running inside the same network with access to a common, shared drive. You will also need a SQL Server 2016+ instance for the Operation Store database which should be accessible from within the nodes.

Logs from across the nodes are shipped to an existing Elasticsearch instance and later visualized in Kibana.

### Operation Store Deployment

The first step of setting up a PowerTree cluster is the SQL Server Operation Store deployment.

This is easily done through invoking a workflow in-memory:

1. Follow the steps outlined in [Command-Line Interface](../service/cli) in order to start a PowerTree session in PowerShell
2. Replace the parameters with your target values

```powershell
$DatabaseName = "OperationStoreLocal";

Invoke-Workflow `
    -Assembly "OperationStore.SqlServer"`
    -Namespace "Grafixoft.Powertree.OperationStore.SqlServer"`
    -Class "DeployOperationStoreSqlServerWorkflow"`
    -FullDirectoryPath (Join-Path $pwd ".\") `
    -Parameters @{`
        # Required
        ConnectionString="Data Source=(LocalDB)\MSSQLLocalDB; Integrated Security=SSPI; Initial Catalog=$DatabaseName";

        # Optional
        DatabaseInitialSize="100MB";`
        DatabaseMaxSize="1000MB";`
        DatabaseFilePath="$env:TEMP\$DatabaseName.mdf";`
        DatabaseFileGrowth="100MB";`
        LogInitialSize="100MB";`
        LogMaxSize="1000MB";`
        LogFilePath="$env:TEMP\$DatabaseName.ldf";`
        LogFileGrowth="100MB";`
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

The deployment workflow example below uses X.509 certificate authentication and assumes the appropriate Service Fabric certificate has been installed in the specified Certificate Store for the current user.

After the PowerTree package is verified, the corresponding application type is registered within the system.
Depending on whether there's an existing instance of the PowerTree application or not, the deployment process chooses to perform a fresh install or upgrade the cluster. 

The upgrade process of PowerTree in the cluster is monitored and executes health checks on every node. If health is in an error or warning state the whole upgrade is rolled back.

All that's needed is to adjust the parameters as nesessary and run the snippet below:

> TODO: PowerShell 5

```powershell
$paramsList = @{`
    # Connection string to a primary node in the cluster
    ClusterConnection = "172.168.0.52:19000";`
    ImageStoreConnectionString = "fabric:ImageStore";`
    CompressPackage = $false;`
    # Temp path to store the package before uploading to Service Fabric
    PackageRootPath = "$env:TEMP\PowerTreePackage";`
    # Path to a folder where the runtime binaries are located
    SourcePath = "C:\PowerTree\";`
    # The certificate store name
    CertificateStoreName = "MY";`
    # The Service Fabric certificate thumbprint
    CertificateThumbprint = "ffffffffffffffffffffffffffffffffffffff";`
    # The common name of the Service Fabric certificate
    CertificateCommonName = "SFCert";`
    # If provided, it is copied in the package output folder
    ExecutionEnvironmentSourcePath = $null;`
    InstanceCount = -1;`
    IsSecuredCluster = $true;`
    ApplicationName = "PowerTreeApp";`
    Version = "1.0.0";`
    ServiceName = "WorkflowHost";`
    EntryPoint = "Workflow.HostManager.exe";`
}

Invoke-Workflow `
    -Assembly "Workflow.ServiceFabricDeployment"`
    -Namespace "Grafixoft.Powertree.Workflow.ServiceFabricDeployment"`
    -Class "RuntimeDeploymentWorkflow"`
    -FullDirectoryPath (Join-Path $pwd ".\Workflow.ServiceFabricDeployment\")`
    -Parameters  $paramsList `
    -TimeoutSec 600
```

### Manual Deployment

Bare metal deployment is the simplest, although most involved, deployment option. 
It consists of copying the PowerTree binaries and their respective configuration to all target compute nodes and setting up a daemon to keep the `Workflow.HostManager` process alive.



