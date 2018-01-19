---
copyright:
  years: 2017, 2018
lastupdated: "2018-01-11"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Generate, Access and Analyze Cloud Foundry Application Logs
This tutorial shows how the [IBM Cloud Log Analysis](https://console.bluemix.net/catalog/services/log-analysis) service can be used to understand and diagnose activities of a Cloud Foundry app that is deployed in the IBM Cloud. You will deploy a Python Cloud Foundry application, generate different types of logs, and search, analyze and visualize those logs by using Kibana, an open-source tool that is offered by the IBM Cloud Log Analysis service.

## Objectives
* Provision the IBM Cloud Log Analysis service
* Deploy a Python Cloud Foundry application
* Generate different types of log entries
* Access application logs
* Search and analyze logs
* Visualize logs

## Introduction

You can use the logging capabilities in the IBM Cloud to understand the behaviour of the cloud platform and the resources that are running in it. No special instrumentation is required to collect the standard out and the standard err logs. For example, you can use logs to provide an audit trail for an application, detect problems in your service, identify vulnerabilities, troubleshoot your app deployments and runtime behavior, detect problems in the infrastructure where your app is running, trace your app across components in the cloud platform, and detect patterns that you can use to preempt actions that could affect your service SLA.

By default, the IBM Cloud offers logging capabilities that are integrated in the Cloud platform:

* Collection of data is automatically enabled for cloud resources. IBM Cloud, by default, collects and displays logs for your apps, apps runtimes, and compute runtimes where those apps run.
* You can search up to 500 MB of logs per day.
* Logs for the last 3 days are stored in Log Search, a component of the Log Analysis service.

In addition, IBM Cloud offers the IBM Cloud Log Analysis service. You can use this service to expand your log collection, log retention, and log search abilities in the IBM Cloud. The [IBM Cloud Log Analysis](https://console.bluemix.net/catalog/services/log-analysis) service provides an easy-to-use interface to logs generated by applications running in the IBM Cloud. In the premium plans, you can also fed logs from sources outside the IBM Cloud for consolidated storage and analysis.

In this tutorial, we are going to take a look at how to generate, access and analyze application logs. The [documentation for the Log Analysis](https://console.bluemix.net/docs/services/CloudLogAnalysis/index.html) service already includes a [tutorial on how to analyze logs for an app deployed in a Kubernetes cluster](https://console.bluemix.net/docs/services/CloudLogAnalysis/containers/tutorials/kibana_tutorial_1.html). Therefore, in this guide, we are going to use a Cloud Foundry application.

## Provision the Log Analysis Service
Applications running in the IBM Cloud generate diagnostic output, i.e. logs, that can be accessed without any additional service. By using the Log Analysis service, it is possible to aggregate logs from various sources and retain them as long as needed. This allows to analyze the "big picture" when required and to troubleshoot more complex situations.

The Log Analysis service is found in the [IBM Cloud Catalog in the DevOps category](https://console.bluemix.net/catalog/?category=devops). Click on it and select the region, organization and space to which you want to provision the service. This should be the same as for the app we are going to deploy in the next step. Once the correct values are set, click on **Create**.

By default the **Lite** plan is selected which allows for 500 MB of daily logs for the past 3 days. The **Premium** plans feature higher data volumes and longer log retention.

## Deploy a Cloud Foundry app
The ready-to-run [code for the logging app is located in this Github repository](https://github.com/IBM-Cloud/application-log-analysis). Clone or download the repository, then push the app to the IBM Cloud.

1. Clone the Github repository:
   ```bash
   git clone https://github.com/IBM-Cloud/application-log-analysis
   cd application-log-analysis
   ```
2. Push the application to the IBM Cloud. You need to be logged in to the region, org and space in which the Log Analysis service was created. Copy and paste these commands one line at a time.
   ```bash
   bx login
   bx target --cf
   bx cf push your-app-name
   ```

## Generate Application Logs
Next, in order to work with application logs, you need to generate them. The application push process above already generated many log entries. Because applications deployed in the IBM Cloud are automatically linked with the Log Analysis service, nothing needs to be done from our side.

1. Visit the web app using the route listed in the **urls** line at the end of the output generated by the push process. The application URL is also shown in the [IBM Cloud dashboard (console)](console.bluemix.net).
2. The application allows you to log a message at a chosen log level. The available log levels are **critical**, **error**, **warn**, **info** and **debug**. The application's logging infrastructure is configured to allow only log entries on or above a set level to pass. Initially, the logger level is set to **warn**. Thus, a message logged at **info** with a server setting of **warn** would not show up in the diagnostic output. The UI allows to change the logger setting for the server log level as well. Try it and generate several log entries.
3. Take a look at the code in the file [**cloud-logging.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/cloud-logging.py). The code contains **print** statements as well as calls to **logger** functions. Printed messages are written to the **stdout** stream (regular output, application console / terminal), logger messages appear in the **stderr** stream (error log).
4. Back in the application, generate several log entries by submitting messages at different levels. Change the server-side log level in-between to make it more interesting.

## Access Application logs
IBM Cloud offers multiple ways of accessing application logs, in our case for a Cloud Foundry app.

1. The first is using the command line. The following displays the recent logs. It is a great method to investigate errors when an app after the push is not starting:
```bash
bx cf logs your-app-name --recent
```
2. The second method is to use the [IBM Cloud console](https://console.bluemix.net). In the overview, navigate to your app, click on its entry to open the details and then go to **Logs**. Current logs are shown with the most recent at the bottom. On the upper right you can search for an entry or filter by log type. Selecting **Application (APP)**
![](images/solution12/Dashboard_LogsFilter.png)
3. The Log Analysis service offers access to the logs. In contrast to the first two, more historial logs from all the apps in a space are available. The logs can be accessed, searched and visualized using a browser-based UI (Kibana dashboard). Using the IBM Cloud dashboard, navigate to **App Name > Logs > View in Kibana** view. [For details see this section in the Log Analysis documentation](https://console.bluemix.net/docs/services/CloudLogAnalysis/kibana/analyzing_logs_Kibana.html#launch_Kibana).   

## Search and Analyze Logs

The Log Analysis / Kibana dashboard, by default shows all available log entries from the past 15 minutes. Most recent entries are shown on the top and automatic refresh is turned off by default. The visible bar chart represents the count of messages per 30 seconds over those 15 minutes. In this section, you will modify what and how much is displayed and save this as **search query** for future use.

1. Available fields that can be displayed and queried are shown on the left. Locate and click on **message > add**.  
![](images/solution12/Dashboard_MessagesAdded.png)
2. If you are seeing logs for more than one application, filter them based on the **app_name_str** field. Instead of **add** use the **+** next to an app name to only see entries for that application or the **-** to exclude the app's logs from the list.   
![](images/solution12/app_name_str.png)
3. Adjust the displayed interval by navigating to the upper right and clicking on **Last 15 minutes**. Adjust the value to **Last 24 hours**.
4. Next to the configuration of the interval is the auto-refresh setting. By default it is switched off, but you can change it.  
5. Below the configuration is the search field. Here you can [enter and define search queries](https://console.bluemix.net/docs/services/CloudLogAnalysis/kibana/define_search.html#define_search). To filter for all logs reported as app errors and containing one of the defined log levels, enter the following:   
```
message:(CRITICAL|INFO|ERROR|WARNING|DEBUG) && message_type_str:ERR
```   
It should look like shown below. The displayed log entries are now filtered based on the search criteria.   
![](images/solution12/SearchForMessagesERR.png)   
6. Store the search criteria for future use by clicking **Save** in the configuration bar. Use **ERRlogs** as name.


## Visualize Logs
Now that you have a query defined, in this section you will use it as foundation for a chart, a visualization of that data. You will first create visualizations and then use them to compose a dashboard.

### Pie Chart as Donut
1. Click on **Visualize** in the left navigation bar.
2. In the list of offered visualizations Locate **Pie chart** and click on it.
3. Select the query **ERRlogs** that you saved earlier.
4. On the next screen, under **Select buckets type**, select **Split Slices**, then for **Aggregation** choose **Filters**. Add 5 filters having the values of **CRITICAL**, **ERROR**, **WARNING**, **INFO** and **DEBUG** as shown here:   
![](images/solution12/VisualizationFilters.png)   
6. Click on **Options** (right to **Data**) and activate **Donut** as view option. Finally, click on the **play** icon to apply all changes to the chart. Now you should see a **Donut Pie Chart** similar to this one:   
![](images/solution12/Donut.png)   
7. Adjust the displayed interval by navigating to the upper right and clicking on **Last 15 minutes**. Adjust the value to **Last 24 hours**.
8. Save the visualization as **DonutERR**.

### Metric
Next, create another visualization for **Metric**.
1. Pick **Metric** from the list of offered visualizations. In step 2, on the left side, click on the name beginning with **[logstash-]**.
2. On the next screen, expand **Metric** to be able to enter a custom label. Add **Log Entries within 24 hours** and click on the **play** icon to update the shown metric.   
![](images/solution12/Metric_LogCount24.png)   
3. Save the visualization as **LogCount24**.

### Dashboard
Once you have added visualizations, they can be used to compose a dashboard. A dashboard is used to display all important metrics and to help indicate the health of your apps and services.
1. Click on **Dashboard** in the left navigation panel, then on **Add** to start placing existing visualizations onto the empty dashboard.
2. Add the log count on the left and the donut chart on the right. Change the size of each component and to move them as desired.
3. Click on the arrow in the lower left corner of a component to view changes to a table layout and additional information about the underlying request, response and execution statistics are offered.
![](images/solution12/DashboardTable.png)   
4. Save the dashboard for future use.

## Expand the Tutorial
Do you want to learn more? Here are some ideas of what you can do next:
* Push the same app again with a different name or use the [app deployed in a Kubernetes cluster](https://console.bluemix.net/docs/services/CloudLogAnalysis/containers/tutorials/kibana_tutorial_1.html). Then, the Log Analysis dashboard (Kibana) will show the combined logs of all apps.
* Filter by a single app.
* Add a saved search and metric only for critical and error events.
* Build a dashboard for all your apps.


## Related Content
* [Documentation for IBM Cloud Log Analysis](https://console.bluemix.net/docs/services/CloudLogAnalysis/index.html)
* [Logging facility for Python](https://docs.python.org/3/library/logging.html)
* [IBM Cloud Log Collection API](https://console.bluemix.net/apidocs/948-ibm-cloud-log-collection-api?&language=node#introduction)
* Kibana User Guide: [Discovering Your Data](https://www.elastic.co/guide/en/kibana/5.1/tutorial-discovering.html)
* Kibana User Guide: [Visualizing Your Data](https://www.elastic.co/guide/en/kibana/5.1/tutorial-visualizing.html)
* Kibana User Guide: [Putting it all Together with Dashboards](https://www.elastic.co/guide/en/kibana/5.1/tutorial-dashboard.html)