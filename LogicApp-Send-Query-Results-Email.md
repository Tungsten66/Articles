# Use a Logic App to send an e-mail with Log Analytics Workspace query results

This walkthrough is to show you have to create a Logic App to send an e-mail if a query against a Log Analytics Workspace returns results.

## Cited Resources:
[Add conditions to control workflow actions in Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-conditional-statement?tabs=consumption) <br/>
[Create a Consumption logic app workflow](https://learn.microsoft.com/en-us/azure/logic-apps/tutorial-build-schedule-recurring-logic-app-workflow#create-a-consumption-logic-app-workflow) <br/>

## Assumptions:

- A Log Analytics Workspace exists and contains tables and data

## Steps:

### Create Logic App

- Sign in to the Azure Portal and click on Create a Resource
-  Search for or navigate to Logic App and click on create
  ![Logic App](images/LogicApp-Send-Query-Results-Email-1.png)
- On the Basics tab enter the information for Subscription, Resource Group, Logic App Name, and Region.
- Plan Type: Consumption
   ![Logic App Plan Type](images/LogicApp-Send-Query-Results-Email-2.png)
- Review + create
- Create

### Configuring Logic App

- Open the Logic App created in last step and click on the Logic app Designer blade on the left side.
- Click on Recurrence
   ![Recurrence](images/LogicApp-Send-Query-Results-Email-3.png)
- I want to get an e-mail if query results are returned once a day so I am selecting Interval: 1 Frequency: Day
- Click on New step
- Search for "Run query and visualize results"
- Select Run query and visualize results from the Actions tab
- 


```powershell
# Replace 'PGJvZHk+VGhlIHF1ZXJ5IHlpZWxkZWQgbm8gZGF0YVRhYmxlLjwvYm9keT4=' with your actual Base64 string
$base64String = 'PGJvZHk+VGhlIHF1ZXJ5IHlpZWxkZWQgbm8gZGF0YVRhYmxlLjwvYm9keT4='

# Decode the Base64 string to ASCII
$asciiString = [System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($base64String))

# Print the result
Write-Host "Decoded ASCII string: $asciiString"
```




## Post Condition:

This can be used for any table to assist in estimating cost over a period of time.  
Several workbooks in Microsoft Sentinel are also available to estimate cost and include additional factors like Microsoft Sentinel benefit for Microsoft 365 E5, A5, F5 and G5 customers.
