---

copyright:
  years: 2016, 2018
lastupdated: "2018-01-09"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}

# Getting started with {{site.data.keyword.openwhisk_short}}


{{site.data.keyword.openwhisk}} is a distributed, event-driven compute service also referred to as Serverless computing or as Function as a Service (FaaS). {{site.data.keyword.openwhisk_short}} runs application logic in response to events or direct invocations from web or mobile apps over HTTP. Events can be provided from {{site.data.keyword.Bluemix}} services like Cloudant and from external sources. Developers can focus on writing application logic, and creating Actions that are executed on demand.
The key benefit of this new paradigm is that you do not explicitly provision servers. Thus, eliminating worry about auto-scaling, high availability, updates, maintenance, and cost for hours of processor time when your server is running but not serving requests.
Your code executes when there is an HTTP call, database state change, or other type of event that triggers the execution of your code.
You get billed by millisecond of execution time (rounded up to the nearest 100 ms), not per hour of VM utilization regardless whether that VM was doing useful work or not.
{: shortdesc}

This programming model is a perfect match for microservices, mobile, IoT, and many other apps. You get inherent auto-scaling and load balancing out of the box, without having to manually configure clusters, load balancers, http plugins, etc. If you happen to run on {{site.data.keyword.openwhisk}}, you also get a benefit of zero administration - meaning that all of the hardware, networking, and software are maintained by IBM. All that you need to do is to provide the code you want to execute and give it to {{site.data.keyword.openwhisk}}. The rest is “magic”. A good introduction into the serverless programming model is available on [Martin Fowler's blog](https://martinfowler.com/articles/serverless.html).

You can also get the [Apache OpenWhisk source code](https://github.com/openwhisk/openwhisk) and run the system yourself.

For more details about how {{site.data.keyword.openwhisk_short}} works, see [About {{site.data.keyword.openwhisk_short}}](./openwhisk_about.html).

You can use the Browser or the CLI to develop your {{site.data.keyword.openwhisk_short}} applications.
Both have similar capabilities for developing applications; the CLI provides more control over your deployment and operations.

## Develop in your Browser
{: #openwhisk_start_editor}

Try out {{site.data.keyword.openwhisk_short}} in your [Browser](https://console.{DomainName}/openwhisk/actions) to create Actions, automate Actions by using Triggers, and explore public packages. 
Visit the [learn more](https://console.{DomainName}/openwhisk/learn) page for a quick tour of the OpenWhisk User Interface.

## Develop by using the CLI
{: #openwhisk_start_configure_cli}

You can use the {{site.data.keyword.openwhisk_short}} command line interface (CLI) to set up your namespace and authorization key.
Go to [Configure CLI](https://console.{DomainName}/openwhisk/cli) and follow the instructions to install it.

## Overview
{: #openwhisk_start_overview}
- [How OpenWhisk works](./openwhisk_about.html)
- [Common uses cases for Serverless applications](./openwhisk_use_cases.html)
- [Setting up and using OpenWhisk CLI](./openwhisk_cli.html)
- [Using OpenWhisk from an iOS app](./openwhisk_mobile_sdk.html)
- [Articles, samples and tutorials](https://github.com/openwhisk/openwhisk-external-resources)
- [Apache OpenWhisk FAQ](http://openwhisk.org/faq)
- [Pricing](https://console.ng.bluemix.net/openwhisk/learn/pricing)

## Programming model
{: #openwhisk_start_programming}
- [System details](./openwhisk_reference.html)
- [Catalog of OpenWhisk provided services](./openwhisk_catalog.html)
- [Actions](./openwhisk_actions.html)
- [Triggers and Rules](./openwhisk_triggers_rules.html)
- [Feeds](./openwhisk_feeds.html)
- [Packages](./openwhisk_packages.html)
- [Annotations](./openwhisk_annotations.html)
- [Web actions](./openwhisk_webactions.html)
- [API Gateway](./openwhisk_apigateway.html)
- [Entity names](./openwhisk_reference.html#openwhisk_entities)
- [Action semantics](./openwhisk_reference.html#openwhisk_semantics)
- [Limits](./openwhisk_reference.html#openwhisk_syslimits)

## {{site.data.keyword.openwhisk_short}} Hello World example
{: #openwhisk_start_hello_world}
To get started with {{site.data.keyword.openwhisk_short}}, try the following JavaScript code example.

```javascript
/**
 * Hello world as an OpenWhisk action.
 */
function main(params) {
    var name = params.name || 'World';
    return {payload:  'Hello, ' + name + '!'};
}
```
{: codeblock}

To use this example, follow these steps:

1. Save the code to a file. For example, *hello.js*.

2. From the {{site.data.keyword.openwhisk_short}} CLI command line, create the Action by entering this command:
    ```
    wsk action create hello hello.js
    ```
    {: pre}

3. Then, invoke the Action by entering the following commands.
    ```
    wsk action invoke hello --blocking --result
    ```
    {: pre}  

    This command outputs:
    ```json
    {
        "payload": "Hello, World!"
    }
    ```
    
    ```
    wsk action invoke hello --blocking --result --param name Fred
    ```
    {: pre}  

    This command outputs:
    ```json
    {
        "payload": "Hello, Fred!"
    }
    ```

You can also use the event-driven capabilities in {{site.data.keyword.openwhisk_short}} to invoke this Action in response to events. Follow the [alarm service example](./openwhisk_packages.html#openwhisk_package_trigger) to configure an event source to invoke the `hello` Action every time a periodic event is generated.

A complete list of [OpenWhisk Tutorials and Samples can be found here](https://github.com/openwhisk/openwhisk-external-resources#sample-applications). In addition to samples, this repository contains links to articles, presentations, podcasts, videos, and other {{site.data.keyword.openwhisk_short}} related resources.

## API Reference
{: #openwhisk_start_api notoc}
* [REST API Documentation](./openwhisk_reference.html#openwhisk_ref_restapi)
* [REST API](https://console.{DomainName}/apidocs/98)

## Related Links
{: #general notoc}
* [Discover: {{site.data.keyword.openwhisk_short}}](http://www.ibm.com/cloud-computing/bluemix/openwhisk/)
* [{{site.data.keyword.openwhisk_short}} on IBM developerWorks](https://developer.ibm.com/openwhisk/)
* [Apache {{site.data.keyword.openwhisk_short}} project website](http://openwhisk.org)
