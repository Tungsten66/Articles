# Microsoft Sentinel resources that I have found useful
In no apparent order

## Workbooks
- Start here - [Commonly used Microsoft Sentinel workbooks](https://learn.microsoft.com/en-us/azure/sentinel/top-workbooks)
- [Data Collection Rule Toolkit](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/create-edit-and-monitor-data-collection-rules-with-the-data/ba-p/3810987) - Create, Edit, and Monitor Data Collection Rules with the Data Collection Rule Toolkit
- [DoD Zero Trust Strategy Workbook](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/accelerating-zero-trust-alignment-with-microsoft-sentinel/ba-p/3918125)
- [Microsoft Sentinel Optimization Workbook](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/introducing-microsoft-sentinel-optimization-workbook/ba-p/3901489) - Cost and Ingestion Optimization, Operational Optimization and Effectiveness, and Management and Acceleration.
- [Conditional Access changes](https://danielchronlund.com/2022/04/21/a-powerfull-conditional-access-change-dashboard-for-microsoft-sentinel/)
- [Maturity Model for Event Log Management (M-21-31) Solution](https://techcommunity.microsoft.com/t5/public-sector-blog/microsoft-sentinel-maturity-model-for-event-log-management-m-21/ba-p/3074336)
  
## Managing Content hub
- [Automate Microsoft Sentinel Content hub updates](https://charbelnemnom.com/automate-microsoft-sentinel-content-hub-updates/)

## Analytics Rules
- [NRT monitor break glass account access](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/how-to-use-microsoft-sentinel-near-real-time-detections/ba-p/2935352#:~:text=Monitor%20break%20glass%20account%20access)
- [Keep track of Cloud Shell use](https://azurecloudai.blog/2020/08/13/azure-sentinel-analytics-rule-to-keep-track-of-cloud-shell/)
  - The KQL in the article does not work but has been updated on Rod Trent's Github repo - [AR-CloudShellExecution.kql](https://github.com/rod-trent/SentinelKQL/blob/master/AR-CloudShellExecution.kql)
- [Azure Active Directory Identity Protection user account enrichments removed: how to mitigate impact](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/azure-active-directory-identity-protection-user-account/ba-p/3695968)
- [Conditional Access policy changes](https://danielchronlund.com/2022/04/13/monitor-conditional-access-with-microsoft-sentinel/)
- [Create an Alert in Sentinel if someone enables Copilot for Security](https://socautomators.substack.com/p/create-an-alert-in-sentinel-if-someone?r=1xuboc&utm_campaign=post&utm_medium=web&triedRedirect=true)

### Managing Analytics Rules
- [How To Enable Microsoft Sentinel Analytics Rules at Scale](https://charbelnemnom.com/set-microsoft-sentinel-analytics-rules-at-scale/)
- [Update Microsoft Sentinel Analytics Rules at Scale](https://charbelnemnom.com/update-microsoft-sentinel-analytics-rules/)

## Training
- [Microsoft Sentinel Ninja](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/become-a-microsoft-sentinel-ninja-the-complete-level-400/ba-p/1246310)

## Data ingestion
[How to configure Security Events collection with Azure Monitor Agent](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-configure-security-events-collection-with-azure-monitor/ba-p/3770719) - This article is helpful when migrating from the legacy Log Analytics agent and the Defender for Servers auto-provisioning experience to Azure Monitoring Agent (AMA).

## Azure Monitor
- [Azure Arc for Servers Monitoring Workbook](https://techcommunity.microsoft.com/t5/azure-arc-blog/azure-arc-for-servers-monitoring-workbook/ba-p/3298791) - Used this to identify network connectivity issues between on-prem and Azure
- [The Ultimate Azure Inventory Dashboard](https://github.com/scautomation/Azure-Inventory-Workbook) - This is great to find orphaned resources
- [Monitoring for an Azure Server Going Offline](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/monitoring-for-an-azure-server-going-offline/ba-p/4027353)
- [New Resource Reporting](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/new-resource-reporting/ba-p/2150155)

## Testing
[How to Generate Microsoft Sentinel Incidents for Testing and Demos](https://rodtrent.substack.com/p/how-to-generate-microsoft-sentinel)

## Service Limits
- [Service limits for Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/sentinel-service-limits)
- [Azure Monitor service limits](https://learn.microsoft.com/en-us/azure/azure-monitor/service-limits)
- [Log Analytics workspaces](https://learn.microsoft.com/en-us/azure/azure-monitor/service-limits#log-analytics-workspaces)

## MMA to AMA Migration
- [AzTS MMA Discovery and Removal Utility](https://github.com/azsk/AzTS-docs/tree/main/MMA%20Removal%20Utility)
- [How to configure Security Events collection with Azure Monitor Agent](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-configure-security-events-collection-with-azure-monitor/ba-p/3770719) - Useful when migrating from legacy Log Analytics agent and the Defender for Servers auto-provisioning experience

## Retention
[Review and Manage Data Table Retention](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Archive-Log-Tool/ArchiveLogsTool-PowerShell) - PowerShell script to view and manage Data Table Retention
[]()
