---

copyright:
  years:  2018, 2021
lastupdated: "2021-03-28"

keywords: LogDNA, IBM, Log Analysis, logging, api

subcollection: Log-Analysis-with-LogDNA

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}
{:external: target="_blank" .external}

 
# Managing views and alerts programmatically
{: #config_api}

You can use the *Configuration REST API* to manage programmatically views and alerts.
{:shortdesc}

- You can use the **POST** method to create a view, or create a view and attach an alert to it.
- You can use the **PUT** method to modify an existing view, and alerts that are attached to views.
- You can use the **DELETE** method to delete a view and associated alerts.

Before you run any automated tasks, consider doing a back up of your account configuration resources. You can use the back up to restore the resources, to the state before any change is applied, if you encounter problems. See [Export the configuration of resources in a logging instance](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-reuse_resource_definitions#export_config_res).
{: tip}


## Before you begin
{: #config_api_work}

### Creating views
{: #config_api_work_create_views}

When you create a view, consider the following information:
- You can create a view with no alerts. 
- You can create a view with 1 or more notification channels. Valid alert notification channels that are supported by the *Configuration* API are: email, webhook, or PagerDuty.
- The response to an API request to create a view includes the ID of the view.
- The `name` body parameter, that defines the name of the view, is always required.
- A category must exist before you create a view. The request fails if a category is not valid or is not specified.
- You can define body parameters to refine the data that is displayed through the view. You must specify 1 or more of the following body parameters: query, apps, levels, hosts, or tags.

When you create a PagerDuty notification channel, you need to do the following:
- You must manually configure the integration of logging with PagerDuty. See [Integrating with PagerDuty](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-pagerduty).
- You must provide logging with the PagerDuty API key. 

After you create a view, check the view in the logging web UI. 
1. Refresh the browser to see the view listed in the *Views* section. 
2. Select the view to display it.
3. Check the data that is available through the view is the one that you expect.

    - If you do not see any data, check that data is being generated for the logs that you are planning to monitor through this view. For example, you can check the **Everything** view and look for events. 

    - If data is being generated, and you still cannot see any data, check the body parameter values that you have defined for the view. One of them might be set to the wrong value and the search criteria does not find any matching events. 

If you try to create a view without defining a category, you get the following error message:

```json
{"details":[{"message":"\"category[0]\" is not allowed to be empty","key":"category[0]"}],"error":"\"category[0]\" is not allowed to be empty","code":"BadRequest","status":"error"}
```
{: screen}

If you try to create a view without a valid category, you get the following error message:

```json
{"error":"Invalid category name(s): CATEGORYNAME","code":"BadRequest","status":"error"}
```
{: screen}

If you try to define a view and you do not define any of the following body parameters, query, hosts, apps, levels, tags, you can get the following error:

```json
{"details":[{"message":"\"value\" must contain at least one of [query, hosts, apps, levels, tags]","key":"value"}],"error":"\"value\" must contain at least one of [query, hosts, apps, levels, tags]","code":"BadRequest","status":"error"}
```
{: screen}

### Modifying views
{: #config_api_work_modify_views}

When you modify a view, consider the following information:
- You must specify the name and the view ID of the view. 
- If you are viewing a view in the logging web UI, the view is not refreshed automatically after you run an API request to modify the view. To refresh the view in the UI, you must navigate to the `Everything` view and back to the view.

An API request to modify a view replaces the existing view definition and associated alerts with your request body data. Any properties that are not specified in your PUT request will be removed.
{: important}

If the `viewid` that you are trying to modify does not exist, a response similar to the following will be returned: 

```json
{"error":"Nothing to configure","code":"BadRequest","status":"error"}
```
{: screen}

### Deleting views
{: #config_api_work_delete_views}

When you delete a view, consider the following information:
- You must specify the ID of the view that you plan to delete.
- You delete the view and all the alerts that are configured for the view.


## API methods
{: #config_api_methods}

The following table outlines the actions that you can run to manage views and alerts programmatically:

| Action                                                                  | Request  | URL                                  |
| ------------------------------------------------------------------------|----------|--------------------------------------|
| Create a view and attach an alert to a view.                            | `POST`   | `<ENDPOINT>/v1/config/view`          |
| Modify an existing view and the alerts that are attached to the view.   | `PUT`    | `<ENDPOINT>/v1/config/view/<VIEWID>` |
| Delete a view and its associated alerts.                                | `DELETE` | `<ENDPOINT>/v1/config/view/<VIEWID>` |
{: caption="Table 1. logging Configuration API endpoints" caption-side="top"}

Where `<VIEWID>` represents the ID of a view.

## Endpoint URL
{: #config_api_endpoint}

Depending on [your account settings](/docs/account?topic=account-service-endpoints-overview), you can use public or private endpoints to manage views and alerts programmatically. For information about endpoints per region, see [API endpoints](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-endpoints#endpoints_api).


## Authentication
{: #config_api_authentication}

When you manage views and alerts programmatically, you must use a service key. Authorization to the logging Configuration API is enforced by using a service key.
{: note} 

A service key is a unique code that is passed in an API request to identify the calling application or user. The service key is specific to an auditing instance. For more information on how to generate a service key, see [Managing service keys](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-service_keys).

Use of the logging Configuration REST API is done by adding a valid service key to the HTTP Authorization request header. You must pass the service key as a header parameter (`-H`) of your requests.

For example, in a cURL request, you must set the `content-type` header as follows:

```
-H 'servicekey: <SERVICE_KEY>'
```
{: codeblock}


## Additional headers 
{: #config_api_headers}

Some additional headers might be required to make successful requests to the API. Those additional headers are:

### content-type
{: #config_api_headers-content-type}

Define the `Content-Type` header to make successful requests to the API. The `content-type` header specifies that the request body is in JSON format.
{: note}

You must pass the `content-type` in the header parameter (`-H`) of your requests.

For example, in a cURL request, you must set the `content-type` header as follows:

```
-H 'content-type: application/json'
```
{: codeblock}



## Body parameters
{: #config_api_parm}

The following fields are body parameters that you can set in API request:

### name (string)
{: #config_api_parm-name}

Specifies the name of the view.

The following table indicates when the `name` parameter is required:

| Action                                                                | Request  | Field required                       |
| ----------------------------------------------------------------------|----------|--------------------------------------|
| Create a view and attach an alert to a view.                          | `POST`   | ![Check mark icon](images/checkmark-icon.svg "Check mark icon indicating required")|
| Modify an existing view and the alerts that are attached to the view. | `PUT`    | ![Check mark icon](images/checkmark-icon.svg "Check mark icon indicating required")|
| Delete a view and its associated alerts.                              | `DELETE` |  |
{: caption="Table 2. Required status per method" caption-side="top"}


### query (string)
{: #config_api_parm-query}

Specifies the search query that is applied to the view.

Check the query in the logging web UI to validate that the data that is displayed through the view matches the data entry that you plan to monitor through that view.
{: tip}

### hosts (array of strings)
{: #config_api_parm-hosts}

Specifies the list of services from which you want to view data.

In the logging web UI, the value that is set for **Source** in the **Line identifiers** section corresponds to the hosts value of the service that generates that log.

For example, to enter multiple hosts, you must separate the hosts with a comma:

```
"hosts": ["is", "event-streams"]
```
{: codeblock}



### apps (array of strings)
{: #config_api_parm-hosts}

Specifies the service instance ID that generates the log.

For example, to enter multiple apps, you must separate the apps with a comma:

```
"apps": ["apps1", "apps2"]
```
{: codeblock}



### channels (array of objects)
{: #config_api_parm-channels}

Specifies the notification channels and trigger conditions that are associated with a view.
- You can configure 1 or more channels per view.
- You can configure any of the following channels through the *Configuration* API: email, webhook, PagerDuty.

```json
"channels": [
    {
      "integration": "email",
      "emails": ["LIST OF EMAILS"],    // array of strings (emails) that are comma-separated
      "triggerlimit": 15,
      "triggerinterval": "5m",
      "immediate": true,
      "terminal": true,
      "operator": "presence",
      "timezone": "TIMEZONE"
    },
    {
      "integration": "webhook",
      "url": "WEBHOOK_URL",
      "triggerlimit": 25,
      "triggerinterval": "30",
      "operator": "presence",
      "immediate": true,
      "terminal": true,
      "method": "post",
      "headers": {
        "X-MY-HEADER": "My Header Value"
      },
      "bodyTemplate": {
        "my_log_lines": "{{ lines }}"
      }
    },
    {
      "integration": "pagerduty",
      "key": "PAGERDUTY_KEY",
      "triggerlimit": 150,
      "triggerinterval": "15m",
      "operator": "absence",
      "immediate": false,
      "terminal": true
    }
  ]
  ```
  {: codeblock}

### category (array of strings)
{: #config_api_parm-category}

Specifies the classification of views. 
- You can include a view in 1 or more categories.

For example, to associate a view to a category named `My category`, you can set it as follows:

```
"category": ["My category"],
```
{: codeblock}



## Examples of using the logging Configuration API
{: #config_api-samples}

The following are examples of how to use the logging Configuration API.

A category must exist before you create a view. In these examples, replace `<MY_CATEGORY>` with your category. 
{: note}

In these examples, `<SERVICE_KEY>` is the [service key](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-service_keys) for your {{site.data.keyword.la_full_notm}} instance. 

### Creating a view
{: #config_api-create-view}

The following sample creates a view.

```
curl https://api.us-south.logging.cloud.ibm.com/v1/config/view \
  -H 'content-type: application/json' \
  -H 'servicekey: <SERVICE_KEY>' \
  -d '{
  "name": "My RC 200",
  "query": "reasonCode:200",
  "hosts": ["myHostName"],
  "category": ["<MY_CATEGORY>"]
}'
```
{: pre}





### Creating a view and attaching an alert
{: #config_api-create-view-alert}

The following sample creates a view and associates an email alert with the view.

```
curl https://api.eu-de.logging.cloud.ibm.com/v1/config/view \
        -H 'content-type: application/json' \
        -H 'servicekey: <SERVICE_KEY>' \
        -d '{
        "name": "My RC 200",
        "query": "reasonCode:200",
        "hosts": ["myHostName"],
        "category": ["TEST"],
      "channels": [
        {
          "integration": "email",
          "emails": ["myemail@ibm.com"],   
          "triggerlimit": 15,
          "triggerinterval": "5m",
          "immediate": true,
          "terminal": true,
          "operator": "presence",
          "timezone": "Europe/London"
        }
      ]
    }'
```
{: pre}



### Modifying a view by adding an alert
{: #api_configuration-mod-view-alert}

The following modifies a view by adding an alert.

```
curl --request PUT \
  --url https://api.us-south.logging.cloud.ibm.com/v1/config/view/<VIEWID> \
  --header 'Content-Type: application/json' \
  -H 'servicekey: <SERVICE_KEY>'  \
-d '{
  "name": "My RC 200",
  "query": "reasonCode:200",
  "hosts": ["myHostName"],
  "category": ["<CATEGORY"],
  "channels": [
    {
      "integration": "email",
      "emails": ["myemail@ibm.com"],
      "triggerlimit": 15,
      "triggerinterval": "5m",
      "immediate": true,
      "terminal": true,
      "operator": "presence",
      "timezone": "Europe/London"
 }
  ]
}'
```
{: pre}




### Deleting a view
{: #api_configuration-del-view}

The following sample deletes a view.

```
curl --request DELETE \
  --url https://api.us-south.logging.cloud.ibm.com/v1/config/view/<VIEWID> \
  -H 'content-type: application/json' \
  -H 'servicekey: <SERVICE_KEY>'  \
```  
{: pre}


