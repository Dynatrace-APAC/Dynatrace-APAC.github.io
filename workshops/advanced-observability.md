summary: Advanced Observability with Dynatrace
id: advanced-observability
categories: Tech Labs
tags: dem
status: Published 
authors: Brandon Neo
Feedback Link: mailto:APAC-SE-Central@dynatrace.com
Analytics Account: UA-175467274-1

# Advanced Observability with Dynatrace
<!-- ------------------------ -->
## Introduction
Duration: 1

This repository contains the hands on for the Advanced Observability Workshop.

### Prerequisites

* Dynatrace SaaS/Managed Account. Get your free SaaS trial [here](https://www.dynatrace.com/trial/).
* AWS account, with the ability to create an EC2 instance from a public AMI. Signup to a free trial [here](https://aws.amazon.com/free/).
* Chrome Browser
* SSH client such as [mobaxterm](https://mobaxterm.mobatek.net/).

### Lab Setup
The following steps are used for this lab:
- Sample Application 
    * Sample App is based on [easyTravel Docker](https://github.com/Dynatrace/easyTravel-Docker)
    * Follow the [Prerequisite Actions](https://github.com/Dynatrace-APAC/Workshop-DEM/tree/master/Prerequisite%20Actions) to create the application that will be used throughout this workshop.

### What You’ll Learn
- Install OneAgent using Ansible CLI 
- Learn Request Attributes for deep code level capture
- Learn Session Properties for advanced dashboarding
- Learn Metric Ingestion for automated baselines across all entities

<!-- ------------------------ -->
## Automate Agent Rollout
Duration: 15

In this exercise, we will deploy the OneAgent to a Linux instance via Ansible.

Based off our [Dynatrace Ansible Github](https://github.com/Dynatrace/Dynatrace-OneAgent-Ansible), you can rollout Dynatrace Oneagent easily across on Linux and Windows Operating Systems with different available configurations and ensures the OneAgent service maintains a running state. It also provides the tasks to interact with the various OneAgent configuration files.

Adapting from an [Ansible example](https://github.com/popecruzdt/dt-ansible), we will be installing that on our host.
Other Ansible playbooks

[Ansible for Linux](https://github.com/popecruzdt/dt-ansible/blob/popecruzdt/dt-oneagent-install-linux.yml)

### Installing Ansible

Use the command below to setup Ansible in your Linux host

`wget -O- https://raw.githubusercontent.com/Dynatrace-APAC/Workshop-Advanced-Observability/master/setup_ansible.sh | bash`

### Download the Ansible playbook

Use the command below to setup Ansible in your Linux host

`wget https://raw.githubusercontent.com/popecruzdt/dt-ansible/popecruzdt/dt-oneagent-install-linux.yml`

Using `more dt-oneagent-install-linux.yml`, we can explore the file to preview the necessary variables that we need to provide.
For the purposes of the lab, we will use only be using Ansible to rollout on a host. But in a real world context, you can run the playbook across multiple hosts or environments. 

```bash
# dynatrace oneagent install on linux
# hosts_group -> inventory group of hosts to execute playbook on
# dt_api_endpoint -> dynatrace environment api endpoint (include trailing /)
# dt_api_token -> dynatrace api install token
# dt_host_group -> dynatrace host group name
# dt_app_log_content_access -> flag to enable or disable log analytics on host (0 or 1)
# dt_infra_only -> flag to set cloud infrastructure monitoring mode on host (0 or 1)
---
-
  hosts: localhost
  name: "dynatrace oneagent install on linux"
  tasks:
    -
      name: "validate ansible execution on linux with sudo access"
      shell:
        cmd: ls /opt
      become: yes
      changed_when: False
    # Check the latest available OneAgent version
    -
```
### Creating the PaaS Token

Go to **Settings > Integration > Platform as a Service** 

Click on **Generate token** and give a token name eg. **Ansible**

Click on **Generate** and **copy** the token by selecting **Reveal token** to use for the next step.

![Deploy](assets/adv-observe/paas-token.png)

###  Running the playbook

Gather and adjust the variables which you can use for the playbook.

**EXAMPLE**

```bash
ansible-playbook \
-e dt_api_token=<PAAS-TOKEN> \
-e dt_host_group=tech-labs \
-e dt_app_log_content_access=1 \
-e dt_infra_only=0 \
-e dt_api_endpoint=https://mou612.managed-sprint.dynalabs.io/e/<DT-ENV>/api/v1/ \
dt-oneagent-install-linux.yml
```
**Paste** the command into your terminal window and tweak the **dt_api_token** and **dt_api_endpoint** variables. Replace <PAAS-TOKEN> from token you copied earlier. You can get your <DT-ENV> from within the Dynatrace environment from the browser. 

![Deploy](assets/adv-observe/dynatrace-env.png)

Example:

```bash
TO DO OUTPUT EXAMPLE
$

```

### Validate the installation in Deployment status

![Deploy](assets/dem/106-Status.jpg)

Reference: https://www.dynatrace.com/support/help/technology-support/operating-systems/linux/

##  Automatic Discovery & Instrumentation

### Start Sample Application

Run the command below to setup the Sample Application Easytravel.

`wget -O- https://raw.githubusercontent.com/Dynatrace-APAC/Workshop-Advanced-Observability/master/setup_easyTravel.sh | bash`

You can check out status by referring to the logs.

`tail -50f nohup.out`

### Explore the Smartscape

While waiting for Easy Travel to start, you can explore Dynatrace and using the Smartscape, Dynatrace will automatically discover the processes and dependencies that comprises the Easy Travel application! 

[4 things](https://www.dynatrace.com/support/help/get-started/4-things-youll-absolutely-love-about-dynatrace/) that you will love about Dynatrace!

![Smartscape](assets/dem/smartscape.png)

<!-- ------------------------ -->
## Session Properties
Duration: 15

In this exercise, we will cover the setting up Session Properties with Request Attributes. These leveraged for deep visibility into all the details of your users’ interactions with your application. 

### Enable Real time updates for Java

Go to **Settings > Server-side service monitoring > Deep Monitoring > Real-time updates**.

![Deploy](assets/adv-observe/real-time-toggle.png)


### Creating Request Attributes

Go to **Settings > Server-side service monitoring > Request Attributes**

Click on **Define a new request attribute** and use the following:

* Name – **Loyalty status**
* Data type – Text (remains unchanged)
* First value (remains unchanged)
* Leave text as-is (remains unchanged)
* Request attribute source - **Web request URL query parameter**
* Select **Capture on server side** on Drop down menu
* Parameter name - **loyalty**
* Click on **Save**

![Request-attribute](assets/adv-observe/request-attribute-1.gif)

Click on **Add new data source** within the **same request attribute** and use the following:

* Request attribute source – **Java method parameter(s)**
* Click on **Select method sources**
* Select **business.backend**
* Search for **BookingService**
* Select **Use the selected class**
* Search for **checkLoyaltyStatus**
* Choose **2:java.lang.String** on Capture Drop down 
* Click on **Save**

![Request-attribute](assets/adv-observe/request-attribute-2.gif)

Click on **Add new data source** within the **same request attribute** and use the following:

* Request attribute source – **Java method parameter(s)**
* Click on **Select method sources**
* Select **business.backend**
* Search for **AuthenticationService**
* Select **Use the selected class**
* Search for **getLoyaltyStatus**
* Choose **Return value** on Capture Drop down 
* Click on **Save**

![Request-attribute](assets/adv-observe/request-attribute-3.gif)

Ensure that right data sources are in below order and click on **save** on the top right.

1. Authentication Service
2. Booking Service
3. URL query parameter

![Request-attribute](assets/adv-observe/request-attribute.png)

### Validate of Loyalty Status Request Attribute

Go to **Transactions and services** and filter on **AuthenticationService**. 

Click on **View requests** and validate the key-value pairs of Loyalty Status under the **Request Attribute** tab. 

![Request-attribute](assets/adv-observe/request-attribute-values.png)

### Define Session Properties

Go to **Applications > easyTravel Angular > Settings > Session and user action properties**

Click on **Add property > Custom defined property** and use the following:

* Expression type – **Server side request attribute**
* RA name – **Loyalty status**
* Display name – **Loyalty status**
* Key – **loyalty_status**
* Toggle on **Store as session property**
* Aggregation type – Last Value
* Click on **Save propery**

![Session-properties](assets/adv-observe/session-prop-1.gif)

Click on **Add property > Custom defined property** and another property with the following:

Expression type – CSS selector
Data type – Double
CSS selector – button[name="payment:pay"]
Display name – Revenue
Key – revenue
Store as session property
Cleanup rule - Pay\s\$(.*?)\.

![Session-properties](assets/adv-observe/session-prop-2.png)

<!-- ------------------------ -->
## Calculated Service Metrics Properties
Duration: 15

In this exercise, we will cover the setting up Calulated Service Metrics with Request Attributes. These leveraged for deep visibility into all the details of your users’ interactions with your application. 

### Creating Request Attributes

Go to **Settings > Server-side service monitoring > Request Attributes**

Click on **Define a new request attribute** and use the following:

* Name – **Revenue**
* Data type – **Double** 
* **Sum of numeric values**
* Leave text as-is (remains unchanged)
* **Check** the Allow this attribute to access unmasked personal data
* Request attribute source - **Web request URL query parameter**
* Select **Capture on both client and server side** on Drop down menu
* Parameter name - **amount**
* Click on **Save**

Click on **Add new data source** within the **same request attribute** and use the following:

* Request attribute source – **Java method parameter(s)**
* Click on **Select method sources**
* Select **business.backend**
* Search for **BookingService**
* Select **Use the selected class**
* Search for **storeBooking**
* Choose **4:java.lang.Double** on Capture Drop down 
* Click on **Save**

Click on **Add new data source** within the **same request attribute** and use the following:

* Request attribute source – **Java method parameter(s)**
* Click on **Select method sources**
* Select **com.dynatrace.easytravel.frontend.data.DataProviderInterface**
* Search for **DataProviderInterface**
* Select **Use the selected class**
* Search for **storeBooking**
* Choose **4:java.lang.Double** on Capture Drop down 
* Click on **Save**

![Session-properties](assets/adv-observe/request-attribute-list.png)

### Validate of Loyalty Status Request Attribute

Go to **Transactions and services** and filter on **BookingService**. 

Click on **View requests** and validate the key-value pairs of Loyalty Status under the **Request Attribute** tab. 

![Request-attribute](assets/adv-observe/request-attribute-values.png)

### Define Calculated Service Metric

Go to **Settings > Server-side service monitoring > Calculated service metrics**

Click on **Create new metric** and use the following:

* Metric name – **Revenue/LoyaltyStatus**
* Metric source – **Request Attribute**
* Request Attribute – **Revenue**
* Unit - **Custom unit**
* Text field - **Dollars**
* Conditions - **Request Attribute 'Loyalty Status' exists**
* Conditions - **Request Attribute 'Revenue' exists**
* Conditions - **Web service name exists**
* **Toggle** - Spilt by dimension
* Dimension value pattern - **{RequestAttribute:LoyaltyStatus}**
* Dimension name - **LoyaltyStatus**

You'll be able to preview the various dimensions across defined services

![Session-properties](assets/adv-observe/calculated-service-metric.png)

<!-- ------------------------ -->
## Dashboards
Duration: 15

Refer to the left navigation bar and go to Dashboards. Dynatrace has now prebuilt dashboard templates such as **Application performance report** and **Real User Monitoring** dashboards.

![Dashboards](assets/adv-observe/dashboards-list.png)

Drill down into each of these dashboards and explore the various dashboard widgets for App Owners and Business Users.

![Session-properties](assets/adv-observe/app-performance-report.png)

<!-- ------------------------ -->

## Feedback
Duration: 3

We hope you enjoyed this lab and found it useful. We would love your feedback!
<form>
  <name>How was your overall experience with this lab?</name>
  <input value="Excellent" />
  <input value="Good" />
  <input value="Average" />
  <input value="Fair" />
  <input value="Poor" />
</form>

<form>
  <name>What did you benefit most from this lab?</name>
  <input value="Understanding Real User Monitoring setup" />
  <input value="Learning Synthetic" />
  <input value="Learning Session Replay" />
  <input value="Learning User Session Query Language" />
</form>

<form>
  <name>How likely are you to recommend this lab to a friend or colleague?</name>
  <input value="Very Likely" />
  <input value="Moderately Likely" />
  <input value="Neither Likely nor unlikely" />
  <input value="Moderately Unlikely" />
  <input value="Very Unlikely" />
</form>

Positive
: 💡 For other ideas and suggestions, please **[reach out via email](mailto:APAC-SE-Central@dynatrace.com?subject=Kubernetes Workshop - Ideas and Suggestions")**.