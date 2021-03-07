# Azure Optimization Engine

The Azure Optimization Engine (AOE) is an extensible solution designed to generate optimization recommendations for your Azure environment. See it like a fully customizable Azure Advisor. Actually, the first custom recommendations use-case covered by this tool was augmenting Azure Advisor Cost recommendations, particularly Virtual Machine right-sizing, with a fit score based on VM metrics and properties. Other recommendations are being added to the tool, not only for cost optimization but also for security, high availability and other [Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/) pillars. You are welcome to contribute with new types of recommendations!

It is highly recommended that you read the whole blog series dedicated to this project, starting [here](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/augmenting-azure-advisor-cost-recommendations-for-automated/ba-p/1339298). You'll find all the information needed to understand the whole solution.

## README index

* [What you can get](#whatyoucanget)
* [Releases](#releases)
* [Architecture](#architecture)
* [Deployment instructions](#deployment)
* [Usage instructions](#usage)
* [Frequently Asked Questions](#faq)

## <a id="whatyoucanget"></a>What you can get ##

A few hours after setting up the engine, you'll get a Power BI dashboard with all Azure optimization opportunities, coming from both Azure Advisor and from custom recommendations included in the engine. These recommendations are then updated every 7 days and you can contribute with your own custom ones if needed. Check below some examples of the Power BI dashboard pages.

### Built-in custom recommendations

Besides collecting **all Azure Advisor recommendations**, AOE includes other custom recommendations that you can tailor to your needs:

* Cost
    * Augmented Advisor Cost VM right-size recommendations, with fit score based on Virtual Machine guest OS metrics (collected by Log Analytics agents) and Azure properties
    * Unattached disks
    * Standard Load Balancers without backend pool
    * Application Gateways without backend pool
    * VMs deallocated since a long time ago (forgotten VMs)
* High Availability
    * Virtual Machine high availability (availability set, managed disks, storage account distribution when using unmanaged disks)
    * Availability Sets structure (fault/update domains count)
* Security
    * Service Principal credentials/certificates without expiration date
* Operational Excellence
    * Load Balancers without backend pool
    * Service Principal credentials/certificates expired or about to expire

### Recommendations overview

![An overview of all your optimization recommendations](./docs/powerbi-dashboard-overview.jpg "An overview of all your optimization recommendations")

### Cost opportunities overview

![An overview of your Cost optimization opportunities](./docs/powerbi-dashboard-costoverview.jpg "An overview of your Cost optimization opportunities")

### Augmented VM right-size overview

![An overview of your VM right-size recommendations](./docs/powerbi-dashboard-vmrightsizeoverview.jpg "An overview of your VM right-size recommendations")

### Fit score history for a specific recommendation

![Fit score history for a specific recommendation](./docs/powerbi-dashboard-fitscorehistory.jpg "Fit score history for a specific recommendation")

## <a id="releases"></a>Releases ##

* 03/2021 - support for suppressions, new recommendations added and deployment improvements
    * Support for recommendations suppressions (exclude, dismiss, snooze)
    * Five new recommendations added
        * **Cost** - Standard Load Balancers without backend pool
        * **Cost** - Application Gateways without backend pool
        * **Security** - Service Principal credentials/certificates without expiration date
        * **Operational Excellence** - Service Principal credentials/certificates expired or about to expire
        * **Operational Excellence** - Load Balancers without backend pool
    * Helper script that checks whether Log Analytics workspaces are configured with the right performance counters (with auto-fix support)
    * Support for multiple Log Analytics workspaces (VM metrics ingestion by Log Analytics agents)
    * Last deployment options are stored locally to make upgrades/re-deployments easier
    * Roles assigned to the Automation Run As Account have now the least privileges needed
    * Recommendations Dismissed/Postponed in Azure Advisor are now filtered
    * Power BI report improvements
    * Several bug fixes
* 01/2021 - solution deployment improvements and several new recommendations added
    * Support for Azure Cloud Shell (PowerShell) deployment
    * Solution upgrade keeps original runbook schedules
    * Eight new recommendations added
        * **Cost** - VMs that have been deallocated for a long time
        * **High Availability** - Availability Sets with a small fault domain count
        * **High Availability** - Availability Sets with a small update domain count
        * **High Availability** - Unmanaged Availability Sets with VMs sharing storage accounts
        * **High Availability** - Storage Accounts containing unmanaged disks from multiple VMs
        * **High Availability** - VMs without Availability Set
        * **High Availability** - Single VM Availability Sets
        * **High Availability** - VMs with unmanaged disks spanning multiple storage accounts
* 12/2020 - added Azure Consumption dimension to cost recommendations and refactored Power BI dashboard
* 11/2020 - support for automated VM right-size remediations and for other Well-Architected scopes, with unmanaged disks custom recommendation
* 07/2020 - [initial release] Advisor Cost augmented VM right-size recommendations and orphaned disks custom recommendation

## <a id="architecture"></a>Architecture ##

The AOE runs mostly on top of Azure Automation and Log Analytics. The diagram below depicts the architectural components. For a more detailed description, please
read the whole blog series dedicated to this project, starting [here](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/augmenting-azure-advisor-cost-recommendations-for-automated/ba-p/1339298).

![Azure Optimization Engine architecture](./docs/architecture.jpg "Azure Optimization Engine architecture")

## <a id="deployment"></a>Deployment instructions ##

### Requirements

* Azure Powershell 4.5.0+
* A user account with Owner permissions over the chosen subscription and enough privileges to register Azure AD applications ([see details](https://docs.microsoft.com/en-us/azure/automation/manage-runas-account#permissions)), so that the Automation Run As Account is granted the required privileges over the subscription (Reader) and deployment resource group (Contributor)
* (Optional) A user account with at least Privileged Role Administrator permissions over the Azure AD tenant, so that the Run As Account is granted the required privileges over Azure AD (Global Reader)

During deployment, you'll be asked several questions. You must plan for the following:

* Whether you're going to reuse an existing Log Analytics Workspace or a create a new one. **IMPORTANT**: you should ideally reuse a workspace where you have VMs onboarded and already sending performance metrics (`Perf` table), otherwise you will not fully leverage the augmented right-size recommendations capability. If this is not possible/desired for some reason, you can still manage to use multiple workspaces (see [Configuring Log Analytics workspaces](./docs/configuring-workspaces.md)).
* An Azure subscription to deploy the solution (if you're reusing a Log Analytics workspace, you must deploy into the same subscription the workspace is in).
* A unique name prefix for the Azure resources being created (if you have specific naming requirements, you can also choose resource names during deployment)
* Azure region

### Installation

The simplest, quickest and recommended method for installing AOE is by using the **Azure Cloud Shell** (PowerShell). You just have to follow these steps:

1. Open Azure Cloud Shell (PowerShell)
2. Run `git clone https://github.com/helderpinto/AzureOptimizationEngine.git azureoptimizationengine`
3. Run `cd azureoptimizationengine`
4. (optional) Run `Connect-AzureAD` - this is required to grant the Global Reader role to the Automation Run As Account in Azure AD
5. Run `.\Deploy-AzureOptimizationEngine.ps1`
6. Input your deployment options and let the deployment finish (it will take less than 5 minutes)

If the deployment fails for some reason, you can simply repeat it, as it is idempotent. The same if you want to upgrade a previous deployment with the latest version of the repo. You just have to keep the same deployment options. _Cool feature_: the deployment script persists your previous deployment options and lets you reuse it! 

If you don't want to use Azure Cloud Shell and prefer instead to run the deployment from your workstation's file system, you must first install the Az Powershell module (instructions [here](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps)). **IMPORTANT**: don't forget to call the deployment script from a PowerShell **elevated prompt** (by the way, Azure Cloud Shell always runs elevated).

If you choose to deploy all the dependencies from your own local repository, you must publish the solution files into a publicly reachable URL. If you're using a Storage Account private container, you must also specify a SAS token (see syntax and example below)

```powershell
.\Deploy-AzureOptimizationEngine.ps1 -TemplateUri <URL to the ARM template JSON file (e.g., https://contoso.com/azuredeploy.json)> [-ArtifactsSasToken <Storage Account SAS token>] [-AzureEnvironment <AzureChinaCloud|AzureUSGovernment|AzureGermanCloud|AzureCloud>]

# Example 1 - Deploying from a public endpoint not requiring authentication
.\Deploy-AzureOptimizationEngine.ps1 -TemplateUri "https://contoso.com/azuredeploy.json"

# Example 2 - Deploying from a Storage Account, using a SAS Token
.\Deploy-AzureOptimizationEngine.ps1 -TemplateUri "https://aoesa.blob.core.windows.net/files/azuredeploy.json" -ArtifactsSasToken "?sv=2019-10-10&ss=bfqt&srt=o&sp=rwdlacupx&se=2020-06-13T23:27:18Z&st=2020-06-13T15:27:18Z&spr=https&sig=4cvPayBlF67aYvifwu%2BIUw8Ldh5txpFGgXlhzvKF3%2BI%3D"
```

## <a id="usage"></a>Usage instructions ##

Once successfully deployed, and assuming you have your VMs onboarded to Log Analytics and collecting all the [required performance counters](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/augmenting-azure-advisor-cost-recommendations-for-automated/ba-p/1457687), we have everything that is needed to start augmenting Advisor recommendations and even generate custom ones! The first recommendations will be available more or less 3h30m after the deployment. In order to see them, you'll need to connect Power BI to the AOE database (see details below). Every week at the same time, AOE recommendations will be updated according to the current state of your environment.

### Validating whether Log Analytics is collecting the right performance counters

To ensure the VM right-size recommendations have all the required data to provide its full value, Log Analytics must be collecting specific performance counters. You can use the [Setup-LogAnalyticsWorkspaces.ps1](.\Setup-LogAnalyticsWorkspaces.ps1) script as a configuration checker/fixer. For more details, see [Configuring Log Analytics workspaces](./docs/configuring-workspaces.md).

### Widening the scope of AOE recommendations - more subscriptions or more workspaces

By default, the Azure Automation Run As Account is created with Reader role only over the respective subscription. However, you can widen the scope of its recommendations just by granting the same Reader role to other subscriptions or, even simpler, to a top-level Management Group.

In the context of augmented VM right-size recommendations, you may have your VMs reporting to multiple workspaces. If you need to include other workspaces - besides the main one AOE is using - in the recommendations scope, you just have to add their workspace IDs to the `AzureOptimization_RightSizeAdditionalPerfWorkspaces` variable (see more details in [Configuring Log Analytics workspaces](./docs/configuring-workspaces.md)).

### Adjusting AOE thresholds to your needs

For Advisor Cost recommendations, the AOE's default configuration produces percentile 99th VM metrics aggregations, but you can adjust those to be less conservative. There are also adjustable metrics thresholds that are used to compute the fit score. The default thresholds values are 30% for CPU (5% for shutdown recommendations), 50% for memory (100% for shutdown) and 750 Mbps for network bandwidth (10 Mbps for shutdown). All the adjustable configurations are available as Azure Automation variables (for example, `AzureOptimization_PerfPercentile*` and `AzureOptimization_PerfThreshold*`).

For all the available customization details, check [Customizing the Azure Optimization Engine](./docs/customizing-aoe.md).

### Visualizing recommendations with Power BI

The AOE includes a [Power BI sample report](./views/AzureOptimizationEngine.pbix) for visualizing recommendations. To use it, you have first to change the data source connection to the SQL Database you deployed with the AOE. In the Power BI top menu, choose Transform Data > Data source settings.

![Open the Transform Data > Data source settings menu item](./docs/powerbi-transformdatamenu.jpg "Transform Data menu options")

Then click on "Change source" and change to your SQL database server URL (don't forget to ensure your SQL Firewall rules allow for the connection).

![Click on Change source and update SQL Server URL](./docs/powerbi-datasourcesettings.jpg "Update data source settings")

If the connection fails at the first try, this might be because the SQL Database was paused (it was deployed in the cheap Serverless plan). At the next try, the connection should open normally.

The report was built for a scenario where you have an "environment" tag applied to your resources. If you want to change this or add new tags, open the Transform Data menu again, but now choose the Transform data sub-option. A new window will open. If you click next in "Advanced editor" option, you can edit the data transformation logic and update the tag processing instructions.

![Open the Transform Data > Transform data menu item, click on Advanced editor and edit accordingly](./docs/powerbi-transformdata.jpg "Update data transformation logic")

### Suppressing recommendations

If some recommendation is not applicable or you want it to be removed from the report while you schedule its mitigation, you can suppress it, either for a specific resource, resource group, subscription or even solution-wide. See [Suppressing recommendations](./docs/suppressing-recommendations.md) for more details.

## <a id="faq"></a>Frequently Asked Questions ##

* **Is the AOE supported by Microsoft?** No, the Azure Optimization Engine is not supported under any Microsoft standard support program or service. The scripts are provided AS IS without warranty of any kind. The entire risk arising out of the use or performance of the scripts and documentation remains with you.

* **Why is my report empty?** Most of the Power BI report pages are configured to filter out recommendations older than 7 days. If it shows empty, just try to refresh the report data.

* **Why is my VM right-size recommendations overview page empty?** The AOE depends on Azure Advisor Cost recommendations for VM right-sizing. If no VMs are showing up, try increasing the CPU threshold in the Azure Advisor configuration... or maybe your infrastructure is not oversized after all!

* **Why are my VM right-size recommendations showing up with so many Unknowns for the metrics thresholds?** The AOE depends on your VMs being monitored by Log Analytics agents and configured to send a set of performance metrics that are then used to augment Advisor recommendations. See more details [here](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/augmenting-azure-advisor-cost-recommendations-for-automated/ba-p/1457687).

* **Why am I getting an SQL timeout whenever I try to refresh the Power BI report after some time?** The default AOE setup deploys the recommendations database in a serverless plan. The database is paused after 1 hour without usage. If you try to connect to SQL in a paused state, it will awake the database but will return a timeout at the first try. If you don't want this to happen, upgrade the database to a non-serverless SKU.

* **Why am I getting values so small for costs and savings after setting up AOE?** The Azure consumption exports runbook has just begun its daily execution and only got one day of consumption data. After one month - or after manually kicking off the runbook for past dates -, you should see the correct consumption data.

* **What is the currency used for costs and savings?** The currency used is the one that is reported by default by the Azure Consumption APIs. It should match the one you usually see in Azure Cost Management.

* **What is the default time span for collecting Azure consumption data?** By default, the Azure consumption exports daily runbook collects 1-day data from 7 days ago. This offset works well for many types of subscriptions. If you're running AOE in PAYG or EA subscriptions, you can decrease the offset by adjusting the `AzureOptimization_ConsumptionOffsetDays` variable. However, using a value less than 2 days is not recommended.

* **Why is AOE recommending to delete a long-deallocated VM that was deallocated just a few days before?** The _LongDeallocatedVms_ recommendation depends on accurate Azure consumption exports. If you just deployed AOE, it hasn't collected consumption long enough to provide accurate recommendations. Let AOE run at least for 30 days to get accurate recommendations.

