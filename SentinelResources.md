# Microsoft Sentinel resources that I have found useful
In no apparent order

## Best Practices
[Best practices for Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/best-practices)

## Workbooks
- Start here - [Commonly used Microsoft Sentinel workbooks](https://learn.microsoft.com/en-us/azure/sentinel/top-workbooks)
- [Data Collection Rule Toolkit](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/create-edit-and-monitor-data-collection-rules-with-the-data/ba-p/3810987) - Create, Edit, and Monitor Data Collection Rules with the Data Collection Rule Toolkit
- [DoD Zero Trust Strategy Workbook](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/accelerating-zero-trust-alignment-with-microsoft-sentinel/ba-p/3918125)
- [Microsoft Sentinel Optimization Workbook](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/introducing-microsoft-sentinel-optimization-workbook/ba-p/3901489) - Cost and Ingestion Optimization, Operational Optimization and Effectiveness, and Management and Acceleration.
- [Conditional Access changes](https://danielchronlund.com/2022/04/21/a-powerfull-conditional-access-change-dashboard-for-microsoft-sentinel/)
- [Maturity Model for Event Log Management (M-21-31) Solution](https://techcommunity.microsoft.com/t5/public-sector-blog/microsoft-sentinel-maturity-model-for-event-log-management-m-21/ba-p/3074336)
- [Attack Surface Reduction Dashboard](https://github.com/Azure/Azure-Sentinel/blob/master/Workbooks/AttackSurfaceReduction.json)

## Analytics Rules
- [NRT monitor break glass account access](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/how-to-use-microsoft-sentinel-near-real-time-detections/ba-p/2935352#:~:text=Monitor%20break%20glass%20account%20access)
- [Keep track of Cloud Shell use](https://azurecloudai.blog/2020/08/13/azure-sentinel-analytics-rule-to-keep-track-of-cloud-shell/)
  - The KQL in the article no longer works but has been updated on Rod Trent's Github repo - [AR-CloudShellExecution.kql](https://github.com/rod-trent/SentinelKQL/blob/master/AR-CloudShellExecution.kql)
- [Azure Active Directory Identity Protection user account enrichments removed: how to mitigate impact](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/azure-active-directory-identity-protection-user-account/ba-p/3695968)
- [Conditional Access policy changes](https://danielchronlund.com/2022/04/13/monitor-conditional-access-with-microsoft-sentinel/)
- [Create an Alert in Sentinel if someone enables Copilot for Security](https://socautomators.substack.com/p/create-an-alert-in-sentinel-if-someone?r=1xuboc&utm_campaign=post&utm_medium=web&triedRedirect=true)
- [Lookback period](https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-custom?tabs=azure-portal#:~:text=The%20allowed%20range%20for%20both%20of%20these%20parameters%20is%20from%205%20minutes%20to%2014%20days)

### Automation
- [How To Enable Microsoft Sentinel Analytics Rules at Scale](https://charbelnemnom.com/set-microsoft-sentinel-analytics-rules-at-scale/)
- [Update Microsoft Sentinel Analytics Rules at Scale](https://charbelnemnom.com/update-microsoft-sentinel-analytics-rules/)
- [Get-GeoFromIpAndTagIncident](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks/Get-GeoFromIpAndTagIncident)
- [Automate Microsoft Sentinel Content hub updates](https://charbelnemnom.com/automate-microsoft-sentinel-content-hub-updates/)
  
## Training
- [Microsoft Sentinel Ninja](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/become-a-microsoft-sentinel-ninja-the-complete-level-400/ba-p/1246310)

## Data ingestion
- [How to configure Security Events collection with Azure Monitor Agent](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-configure-security-events-collection-with-azure-monitor/ba-p/3770719) - This article is helpful when migrating from the legacy Log Analytics agent and the Defender for Servers auto-provisioning experience to Azure Monitoring Agent (AMA).
- [Use Windows Event Forwarding to help with intrusion detection](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/use-windows-event-forwarding-to-assist-in-intrusion-detection)
- [Sample data collection rules (DCRs) in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-samples)  - Great reference when consolidating DCRs
- [Structure of a data collection rule in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-structure) - Use this when manually modifying a DCR and need to understand the structure

## Azure Monitor
- [Azure Arc for Servers Monitoring Workbook](https://techcommunity.microsoft.com/t5/azure-arc-blog/azure-arc-for-servers-monitoring-workbook/ba-p/3298791) - Used this to identify network connectivity issues between on-prem and Azure
- [The Ultimate Azure Inventory Dashboard](https://github.com/scautomation/Azure-Inventory-Workbook)
- [Azure Orphaned Resources v3.0](https://github.com/dolevshor/azure-orphan-resources/tree/main) - This is great to find orphaned resources
- [Monitoring for an Azure Server Going Offline](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/monitoring-for-an-azure-server-going-offline/ba-p/4027353)
- [New Resource Reporting](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/new-resource-reporting/ba-p/2150155)
- [Log data ingestion time in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-ingestion-time)

## Testing
[How to Generate Microsoft Sentinel Incidents for Testing and Demos](https://rodtrent.substack.com/p/how-to-generate-microsoft-sentinel)

## Service Limits
- [Service limits for Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/sentinel-service-limits)
- [Azure Monitor service limits](https://learn.microsoft.com/en-us/azure/azure-monitor/service-limits)
- [Log Analytics workspaces](https://learn.microsoft.com/en-us/azure/azure-monitor/service-limits#log-analytics-workspaces)

## MMA to AMA Migration
- [MMA/OMS Discovery and Removal Utility](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-mma-removal-tool)
- [How to configure Security Events collection with Azure Monitor Agent](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-configure-security-events-collection-with-azure-monitor/ba-p/3770719) - Useful when migrating from legacy Log Analytics agent and the Defender for Servers auto-provisioning experience

## Azure Data Explorer
 - [Compare ingestion methods](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-data-overview#compare-ingestion-methods)

## Retention
- [Review and Manage Data Table Retention](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Archive-Log-Tool/ArchiveLogsTool-PowerShell) - PowerShell script to view and manage Data Table Retention
-  [Log Analytics workspace data export in Azure Monitor - Supported Tables](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/logs-data-export?tabs=portal#supported-tables)
- [Integrate Azure Data Explorer as Long-Term Log Retention for Microsoft Sentinel - Automation](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/automation-integrate-azure-data-explorer-as-long-term-log/ba-p/2512703)

## Private Endpoints
Using private endpoints for your resources enables you to increase security for the virtual network by enabling you to block exfiltration of data from the virtual network.
- [Azure Monitor Private Link Scope (AMPLS)](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/private-link-security)
- [When using AMA with AMPLS, all of your Data Collection Rules must use Data Collection Endpoints. Those DCE's must be added to the AMPLS configuration using private link.](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-network-configuration?tabs=PowerShellWindows#:~:text=When%20using%20AMA%20with%20AMPLS%2C%20all%20of%20your%20Data%20Collection%20Rules%20much%20use%20Data%20Collection%20Endpoints.%20Those%20DCE%27s%20must%20be%20added%20to%20the%20AMPLS%20configuration%20using%20private%20link)
- [Create an Azure DNS Private Resolver](https://learn.microsoft.com/en-us/azure/dns/private-resolver-hybrid-dns#create-an-azure-dns-private-resolver) - Create an Azure DNS Private Resolver with an inbound endpoint
- Use Public DNS zone forwarders in hybrid scenario to configure ADDS DNS conditional fordwarders [Azure Private Endpoint private DNS zone values](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#management-and-governance)
- [Event Hubs](https://learn.microsoft.com/en-us/azure/event-hubs/private-link-service)
- [Private endpoints for Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/security-network-private-endpoint)

## KQL
- [KQL Cheat Sheet](https://github.com/marcusbakker/KQL/blob/master/kql_cheat_sheet_v01.pdf)
- [KQL Search](https://www.kqlsearch.com/)
- 

[]()
