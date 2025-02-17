---

copyright:
  years:  2018, 2021
lastupdated: "2021-03-28"

keywords: LogDNA, IBM, Log Analysis, logging, config agent

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

# Logging agent
{: #logdna_agent}

The logging agent is responsible for collecting and forwarding logs to your {{site.data.keyword.la_full_notm}} instance. After you provision an instance of {{site.data.keyword.la_full}}, you must configure a logging agent for each log source that you want to monitor.
{:shortdesc}

* You can configure a logging agent to connect to an {{site.data.keyword.la_full_notm}} instance through the public network or through the private network. By default, the agent connects through the public network. To connect to {{site.data.keyword.cloud}} services over a private network, you must have access to the classic infrastructure and [enable virtual routing and forwarding (VRF)](/docs/account?topic=account-vrf-service-endpoint) and connectivity to service endpoints for your account.
* The logging agent authenticates by using the logging ingestion key and opens a secure web socket to the {{site.data.keyword.la_full_notm}} ingestion servers.
* The logging agent for Kubernetes automatically collects STDOUT and STDERR logs.
* By default, the logging agent monitors all files with extension *.log*, and extensionless files under */var/log/*.
* The logging agent tails for new log data, and looks for new files that are added to the logging directories that the agent monitors.


## Logging agent for Kubernetes clusters
{: #logdna_agent_image_kube}

logging agent images for Kubernetes clusters are public images that are available in {{site.data.keyword.cloud_notm}} through the [{{site.data.keyword.registrylong_notm}}](/docs/Registry?topic=Registry-getting-started) service. 

* The logging agent images are hosted in the {{site.data.keyword.registrylong_notm}} global repository `icr.io/ext/logdna-agent`.

* {{site.data.keyword.registrylong_notm}} provides a multi-tenant, highly available, scalable, and encrypted private image registry that is hosted and managed by {{site.data.keyword.IBM_notm}}. 

* The {{site.data.keyword.registrylong_notm}} includes *Vulnerability Advisor* features that scan for potential security issues and vulnerabilities. Vulnerability Advisor checks for vulnerable packages in specific Docker base images, and known vulnerabilities in app configuration settings. When vulnerabilities are found, information about the vulnerability is provided. You can use this information to resolve security issues so that containers are not deployed from vulnerable images. 

To get details about the logging agent images, see [Getting information about logging agent images ](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-logdna_agent_image).


### Understanding image tags
{: #logdna_agent_image_kube_tags}

The tag that is associated to a logging image indicates whether the logging agent is automatically updated. 
{: important}

A tag consists of multiple parts:

```
X.Y.Z-<date>.[hash]
```
{: codeblock}

Where

- `X` represents the major version of an image.
- `Y` represents the minor version of an image.
- `Z` represents an incremental ID that determines the latest patched minor version.
- `<date>` represents the date that the image is built and available. The format is `YYYYMMDD`.
- `[hash]` represents the digest (manifest) of the container image. It is a unique `SHA-256` hash.


The following table outlines the tagging convention adopted and the agent update behaviour:

| Tag | logging agent auto-update enabled | More info |
|-----|----------------------------------|-----------|
| `X` |  YES  | The logging agent auto-updates when a new minor version releases. </br>The logging agent does not update to a new major version, as these updates may require configuration changes. |
| `X.Y`  | YES | The logging agent auto-updates when a new patch version is released. |
| `X.Y.Z` | YES | The logging agent auto-updates when a new vulnerability fix is released. The agent code does not change, but the included libraries have vulnerability fixes. |
| `X.Y.Z-<date>.[hash]` | NO | The logging agent never updates. If you use this tag, make sure you are watching for new agent releases that have vulnerability fixes. |
{: caption="Table 1. logging agent tags explained" caption-side="top"}


For highly regulated environments, use the tag `X.Y.Z-<date>.[hash]`.
{: important}


### Stable and latest tags (deprecated)
{: #logdna_agent_image_tags_1}

For the `V1` and `V2` agent versions, you can also find the tags `stable` and `latest`. When you use any of these tags to configure the logging agent, notice that the logging agent will automatically update to the latest version of the agent.

- The tag `latest` refers to the most recent logging agent 1.Y image.
- The tag `stable` refers to the most recent logging agent 2.Y image.

The tags `stable` and `latest` will be deprecated in June 2021.
{: important}



### Image versions
{: #logdna_agent_image_kube_versions}


The following table outlines the logging agent versions that are available to configure for a Kubernetes cluster:

| Kubernetes cluster             | logging agent V3             | logging agent V2       | logging agent V1                              |
|--------------------------------|-----------------------------|-----------------------|----------------------------------------------|
| `Standard Kubernetes cluster`  | ![Checkmark icon](../images/checkmark-icon.svg) | ![Checkmark icon](../images/checkmark-icon.svg) | ![Checkmark icon](../images/checkmark-icon.svg) |
| `OpenShift Kubernetes cluster` | ![Checkmark icon](../images/checkmark-icon.svg) | ![Checkmark icon](../images/checkmark-icon.svg) | `Not available`                              |
{: caption="Table 2. logging agent versions for Kubernetes clusters" caption-side="top"}


The logging agent v2 is available only for Kubernetes 1.9+.
{: important}


### Choosing a version
{: #logdna_agent_image_kube_choose_version}

When you configure the logging agent, you can choose any of the following options:
- You can use the default YAML that is provided. Choose by region and by type of account. The default configuration pulls the image `icr.io/ext/logdna-agent:stable`.
- You can use an existing YAML file and change the `image:tag` value in your existing YAML file to pull a specific image from the registry.

If you have a highly regulated environment, you can customize the YAML file. You can modify the YAML file so that it pulls from the {{site.data.keyword.registrylong_notm}} global repository `icr.io/ext/` the image that you specify, for example, `image: icr.io/ext/logdna-agent:X.Y.Z-<date>.[hash]`. Consider keeping a copy of the customized YAML file in a version control system. 
{: important}


### Connecting a logging agent with a logging instance 
{: #logdna_agent_image_kube_connect}

The logging agent is responsible for collecting and forwarding system-level, and file-based logs to your {{site.data.keyword.la_full_notm}} instance. 

To connect your Kubernetes cluster to send logs to your {{site.data.keyword.la_full_notm}} instance, you must install a *logging-agent* pod on each node of your cluster. 

- The logging agent reads log files from the pod where it is installed, and forwards the log data to your logging instance.

- The logging agent collects STDOUT, STDERR, logs with the extension `*.log`, and extensionsless files that are stored in the `/var/log` directory of your pod. By default, logs are collected from all namespaces, including `kube-system`, and automatically forwarded to the {{site.data.keyword.la_full_notm}} service.

- To connect an agent to a standard Kubernetes cluster, see [Connecting a logging agent for a standard Kubernetes cluster](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-config_agent_kube_cluster).

- To connect an agent to an OpenShift Kubernetes cluster, see [Connecting a logging agent for an OpenShift Kubernetes cluster](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-config_agent_os_cluster).



### Detaching a logging agent from a cluster
{: #logdna_agent_detach}

To stop your Kubernetes cluster from sending logs to your {{site.data.keyword.la_full_notm}} instance, you must remove the logging agent from your cluster. [Learn more](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-detach_agent).

| Platform                       | How to install and configure |
|--------------------------------|------------------------------|
| `Standard Kubernetes cluster`  | [Detaching a logging agent from a standard Kubernetes cluster](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-detach_agent#detach_agent_kube) |
| `OpenShift Kubernetes cluster` | [Detaching a logging agent from an Openshift Kubernetes cluster](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-detach_agent#detach_agent_os) |
{: caption="Table 10. Detaching a logging agent from a cluster" caption-side="top"}


## Logging agent for Linux
{: #logdna_agent_image_linux}



### Image versions
{: #logdna_agent_image_linux_versions}

logging agent images for Linux are public images that are available in the logging repo `https://repo.logdna.com`.

The following table outlines the agent versions that are available:

| Kubernetes cluster                    | logging agent V1                                    | logging agent V2                                   |
|---------------------------------------|----------------------------------------------------|---------------------------------------------------|
| `Linux                      `         | ![Checkmark icon](../images/checkmark-icon.svg)  | `Not available`                                   |
{: caption="Table 3. logging agent versions for Linux" caption-side="top"}



### Connecting a logging agent with a logging instance 
{: #logdna_agent_image_linux_connect}

The logging agent is responsible for collecting and forwarding system-level, and file-based logs to your {{site.data.keyword.la_full_notm}} instance. 

To connect your Linux server to send logs to your {{site.data.keyword.la_full_notm}} instance, you must install a *logging-agent*. 

- The logging agent reads log files from */var/log*, and forwards the log data to your logging instance. 

- To connect an agent to a Linux platform, see [Connecting a logging agent for Linux Ubuntu or Debian](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-config_agent_linux).




## Configuring a logging agent
{: #logdna_agent_configure}

You can customize a logging agent by configuring parameters for Linux agents, or environment variables for Kubernetes agents.

### Linux: configuration parameters for logging agent V1
{: #logdna_agent_configure_linux_v1}

| Parameter | Description                                          |
|-----------|------------------------------------------------------|
| `tags`    | Define tags to group hosts automatically into dynamic groups. |
| `logdir`  | Define custom paths that you want the agent to monitor. </br>Separate multiple paths by using commas. You can use glob patterns. You can configure specific files. Enter glob patterns by using double quotation marks. | 
| `exclude` | Define the files that you do not want the logging agent to monitor. **Note:** These files can be located in any of the paths that are defined through the logdir parameter. </br>Separate multiple files by using commas. You can use glob patterns. You can configure specific files. |
| `exclude_regex` | Define regex patterns to filter out any lines that match the pattern. Do not include leading and trailing `/`. | 
| `hostname` | Define the hostname. This value overrides the operating system hostname. |
| `autoupdate` | Set to `1` to update the agent automatically when the public repo agent definition is updated. Set to `0` to disable this feature. |
{: caption="Table 6. Configuration options for the logging agent V1" caption-side="top"}


### Standard Kubernetes clusters: environment variables for logging agent V2
{: #logdna_agent_configure_std_kube_v2}

| Environment variable     | Description                                           |  Default value                         | Sample value          |
|--------------------------|-------------------------------------------------------|----------------------------------------|-----------------------|
| `DEFAULT_CONF_FILE`      | Default configuration file.                           | `/etc/logdna/config.yaml`              |                       |
| `LOGDNA_PLATFORM`        | Log source type.                                      | `k8s`                                  |                       |
| `LOGDNA_INGESTION_KEY`   | Reference to the logging ingestion key.                | secretKeyRef                           |                       |
| `LOGDNA_HOST`            | LogDNA ingestion endpoint.                            |                                        | `logs.us-south.logging.cloud.ibm.com` |
| `LOGDNA_API_HOST`        | API LogDNA ingestion endpoint.                        |                                        | `api.us-south.logging.cloud.ibm.com`   |
| `LOGDNA_ENDPOINT`        | Ingestion log path.                                   | `/logs/agent/`                         |                       |
| `LOGDNA_HOSTNAME`        | Log Source name.                                      |                                        | `MyCluster`                            | 
| `LOGDNA_LOG_DIRS`        | Defines custom paths that you want the agent to monitor. </br>Separate multiple paths by using commas. </br>You can use glob patterns. Use double quotation marks to add a globe pattern.   | `/var/log/`   | `/output/,/mylogs/myapplogs/` |
| `LOGDNA_INCLUSION_RULES` | Custom rules that you can define to configure what log files to monitor. </br>These files can be located in any of the paths that are defined through the logdir parameter. </br>You can use glob patterns. For more information, see [Glober rules](https://github.com/CJP10/globber){: external}   |  | `*.json,*.test` |
| `LOGDNA_INCLUSION_REGEX_RULES` | Regex custom rules that you can define to configure what log files to monitor. For more information, see [regex syntax](https://docs.rs/regex/1.2.1/regex/#syntax){: external} </br>These files can be located in any of the paths that are defined through the logdir parameter.  |  |  |
| `LOGDNA_EXCLUSION_RULES` | Custom rules that you can define to configure what log files to exclude from being monitored. </br>You can use glob patterns. For more information, see [Glober rules](https://github.com/CJP10/globber){: external}  | | |
| `LOGDNA_EXCLUSION_REGEX_RULES` | Regex custom rules that you can define to configure what log files to exclude from being monitored. |  | `/var/log/containers/**,/var/log/pods/**`  |
| `LOGDNA_USE_SSL`          | Boolean that defines whether TLS 1.2 should be used when the agent sends logs to the logging instance. </br>The default value is set to `true`.  | `true` | `true` |
| `LOGDNA_USE_COMPRESSION`  | Boolean that defines whether compression is enabled when the agent sends logs to the logging instance. </br> The default value is set to `true`. | `true` | `true` |
| `LOGDNA_GZIP_LEVEL`       | Compression level for gzip. </br>Valid values are: `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9` </br>When you set this variable to `1`, you are configuring the agent to use the fastest compression speed but at a lower ratio. When you set this variable to `9`, you are configuring the agent to use the highest compression ratio but at a lower speed.  | `2` | `6` |
| `LOGDNA_TAGS` | Define tags to group hosts automatically into dynamic groups. |  | `production,serviceA` |
{: caption="Table 7. Tags that are available for the logging agent V2" caption-side="top"}

### Standard Kubernetes clusters: environment variables for logging agent V1
{: #logdna_agent_configure_std_kube_v1}

| Environment variable     | Description                                           | Default value                          | Sample value          |
|--------------------------|-------------------------------------------------------|----------------------------------------|-----------------------|
| `DEFAULT_CONF_FILE`      | Default configuration file.                           | `/etc/logdna/config.yaml`              |                       |
| `LOGDNA_PLATFORM`        | Log source type.                                      | `k8s`                                  |                       |
| `LOGDNA_INGESTION_KEY`   | Reference to the logging ingestion key.                | secretKeyRef                           |                       |
| `LDLOGHOST`              | LogDNA ingestion endpoint.                            |                                        | `logs.us-south.logging.cloud.ibm.com` |
| `LDAPIHOST`              | API LogDNA ingestion endpoint.                        |                                        | `api.us-south.logging.cloud.ibm.com`   |
| `LDLOGPATH`              | Ingestion log path.                                   | `/logs/agent/`                         |                       |
| `LOGDNA_HOSTNAME`        | Log Source name.                                      |                                        | `MyCluster`                            | 
| `LOG_DIRS`            | Defines custom paths that you want the agent to monitor. </br>Separate multiple paths by using commas. </br>You can use glob patterns. Use double quotation marks to add a globe pattern.  | `var/log` | `/output/,/mylogs/myapplogs/` |
| `LOGDNA_INCLUDE`     | Custom rules that you can define to configure what log files to monitor. </br>These files can be located in any of the paths that are defined through the logdir parameter. </br>You can use glob patterns. For more information, see [Glober rules](https://github.com/CJP10/globber){: external}  |  | `*.json,*.test` |
| `LOGDNA_INCLUDE_REGEX` | Regex custom rules that you can define to configure what log files to monitor. For more information, see [regex syntax](https://docs.rs/regex/1.2.1/regex/#syntax){: external} </br>These files can be located in any of the paths that are defined through the logdir parameter.  | | |
| `LOGDNA_EXCLUDE`     | Custom rules that you can define to configure what log files to exclude from being monitored. </br>These files can be located in any of the paths that are defined through the logdir parameter. </br>Separate multiple files by using commas. You can use glob patterns. You can configure specific files. | | |
| `LOGDNA_EXCLUDE_REGEX` | Regex custom rules that you can define to configure what log files to exclude from being monitored. | | `/var/log/containers/*,/var/log/pods/*`  |
| `LDLOGSSL` | Boolean that defines whether TLS 1.2 should be used when the agent sends logs to the logging instance. </br>The default value is set to `true`. | `true` | `true` |
| `COMPRESS` | Boolean that defines whether compression is enabled when the agent sends logs to the logging instance. </br> The default value is set to `true`. | `true` | `true` |
| `GZIP_COMPRESS_LEVEL` | Compression level for gzip. </br>Valid values are: `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9` </br>When you set this variable to `1`, you are configuring the agent to use the fastest compression speed but at a lower ratio. When you set this variable to `9`, you are configuring the agent to use the highest compression ratio but at a lower speed.  | `2` | `6` |
| `LOGDNA_TAGS` | Define tags to group hosts automatically into dynamic groups. |  | `production,serviceA`  |
{: caption="Table 8. Tags that are available for the logging agent V2" caption-side="top"}



### Configure tags to group data
{: #logdna_agent_tags}

You can configure tags at the agent level so that all lines that are sent by this agent can be grouped automatically into a group when you filter data in a view.

* You can define multiple tags. 
* You separate tags by using commas. 
* The maximum number of characters that you can set to define multiple tags is 80 characters.

| Platform                       | How to install and configure |
|--------------------------------|------------------------------|
| `Kubernetes cluster`           | [Adding tags to logs from a Kubernetes cluster](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-adding_tags#adding_tags_kube) |
| `Linux Ubuntu or Debian`       | [Adding tags to logs from Linux Ubuntu or Debian](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-adding_tags#adding_tags_linux) |
{: caption="Table 9. Adding tags" caption-side="top"}


### Exclude log files that are monitored by the agent
{: #logdna_agent_exclude}

You can configure a logging agent to exclude logs that you do not want to monitor through the logging UI. 

* You can exclude files that are located in any of the paths that are defined through the **logdir** parameter in a Linux system or the **LOGDNA_EXCLUDE** variable in a Kubernetes cluster. 
* You can configure multiple files. You separate multiple files by using commas. 
* You can use glob patterns to define what you want to exclude. 
* You can configure specific files.

For more information, see [Excluding log files through the logging agent](/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-exclude_logs_from_agent).





