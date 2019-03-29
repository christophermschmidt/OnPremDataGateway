	Welcome to this week’s Power BI tip of the week! This week we’ll be taking a slight break away from building something in DAX, and focusing on monitoring of the on prem data gateway.

Many customers are using on prem data gateways, whether for Power BI, Flow, Power Apps, Logic Apps or Azure Analysis Services. Frequently, however, the monitoring portion of that data gateway is overlooked. As data refreshes and queries against the data source occur, care must be taken to see the effect on memory, networking, CPU, and other key metrics. This ensures that the gateway is operating at peak efficiency and that the transfer of data is not bottlenecked. When failures occur alerts can be set and a quick glance on what was happening on the server can be viewed. Was the network saturated at the time? Was the machine out of memory? Are there enough CPU’s available to support the transformations needed? Was the mashup engine (Query Editor) under duress? Regardless of the owner, a failure of the gateway is viewed by the user as an issue with the report. So it’s prudent as a team to ensure that if the gateway causes the failure, proactive notification occurs in order to prevent issues from plaguing the user. Ideally the BI team should have as much insight into the performance of the gateway as the monitoring team (assuming there’s a separate team).

If the organization is already using Log Analytics to monitor additional Azure services, the best practice would be to use Microsoft Monitoring Agent to load the perfmon counters and event data into Log Analytics. If that’s not available, or not plausible for the analytics team to configure, alternatively the option remains to roll your own using a little bit of PowerShell to save the counters as a csv. This week’s tip will focus on the using the Microsoft Monitoring agent approach. Note that this is the same process for configuring any machine to send data to Log Analytics, so if the organization has a formalized process for this an option would be to give that team the server name and have them configure. When combined with other Azure services streaming data to LA, this provides a centralized place to pull the metrics out of. To configure this, let’s start by navigating into Log Analytics and clicking on Advanced Settings, and capture the WorkspaceID and the PrimaryKey:

 


Next, ensure that the server is set to TLS 1.2 protocol for communication to the Log Analytics service. Follow the rest of the steps in the documentation to modify the appropriate registry keys, and install the MMA service. Once this is complete, open up the Log Analytics repository you configured and simply query for the data you’re interested in. PerformanceCounters are tracked in the Perf table. If one wanted to see the performance counters that were being collected, the below query would show this:

Perf
| distinct CounterName


By default, only a few counters are tracked. Chances are you’ll want to collect additional counters, as well, so navigate over to the advanced setting section previously that provided the workspaceID and primaryKey.  Under Data, select “WindowsPerformanceCounters”, simply add in the additional performance counters that you want to collect:

 

While you’re here, click on the Windows Event Logs section and add in collectors for the Application and System logs. This will provide the ability to query for system errors and application errors from Log Analytics. There are options here, so check on whether you want to gather all events or just errors and warnings:

 


There is a CustomLogs section as well, so this could potentially be expanded to pull in the gateway error logs and service logs instead of needing to remote to the machine. I’m happy to work with someone on getting this set up if there’s interest. For now, Save your changes and then wait a few minutes for data to begin flowing into the repository.  In an environment with multiple gateways, setting this up for each machine a comprehensive view of all of the gateways in the environment could be obtained relatively easily. Click back over to the query view, and then start analyzing your gateway. All of the perfmon counters are captures in the Perf table, and the EventLog data for both applications and System is kept in the Event table. For example, if I wanted to visualize all of the Perf counters over the past 24 hours, a query that could be used to export this to Power BI (we are PBI developers after all 😊) . In the top right of the screen, select Export, and then click the last “Export to Power BI (M Query) option:

 


Now go open up Power BI Desktop and paste the query. In the sample template provided, I’ve gone ahead and parameterized the Log Analytics Endpoint and the TimeSpan to pull data back from. Once this is configured, if you would prefer to not log in to Log Analytics to get the workspaceID and query string, it can also be obtained via PowerShell. Simply open up a PowerShell window and execute the following: 

import-module Az

#login to azure
Connect-AzAccount

$resourceGroupName = "theNameOfTheResourceGroup"
$workspaceName = "theNameOfTheWorkspace"


$subscriptionID = (Get-AzContext).Subscription.Id
$uri = "https://portal.loganalytics.io/subscriptions/{0}/resourcegroups/{1}/workspaces/{2}" -f $subscriptionId, $resourceGroupName, $workspaceName
$uri 


Once this is configured, by loading and creating the model we are able to quickly drill into performance. For example, if I wanted to see the available memory on the machine, selecting Memory, then AvailableMBytes, gives a line chart showing the memory on the machine. With the computer slicer in the top right either all machines could be viewed together or filtered down depending on the counter being visualized:

 


Obviously this could be broken out and customized as needed. Perhaps rather than having 1 line chart they could be broken out into a visual for each important perfmon counter. That way slicers don’t need to be selected in order to see the counters for that machine. “Server health at a glance”, as they say. 😊 



 
