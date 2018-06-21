---
author: Kent Hua
comments: true
date: "2016-07-18T10:00:00Z"
modified_time: "2016-07-18T11:00:19.397-07:00"
tags: [openshift, cdk, osx, local, docker]
title: OpenShift Container Platform Local (formerly cdk) v2.1
---

Red Hat provides you with a local OpenShift 3 environment within it's container development kit, known as [OpenShift Container Platform Local](http://developers.redhat.com/products/cdk/overview/).  Reference the [instructions](http://developers.redhat.com/products/cdk/docs-and-apis/) for your specific platform.  This is all available to you with a free Red Hat Developers subscription.

Once you have it configured (you can specify your Red Hat credentials using environment variables, `SUB_USERNAME` and `SUB_PASSWORD`), just fire it up with:

{{< highlight shell >}}
vagrant up
{{< / highlight >}}

Follow instructions at the end of the command for access to your local OpenShift environment running on Red Hat Enterprise Linux.

To see the vagrant exportable environment variables:

{{< highlight shell >}}
vagrant service-manager env
{{< / highlight >}}

To apply these environment variables to your local environment (for OSX): 

{{< highlight shell >}}
eval "$(vagrant service-manager env)"
{{< / highlight >}}

Or for all environment variables:

{{< highlight shell >}}
eval "$(vagrant service-manager env docker)"
{{< / highlight >}}


Lastly, suppose you want to run docker commands locally within the shell of your [local operating system](https://access.redhat.com/documentation/en/red-hat-container-development-kit/2.1/getting-started-guide/#preparing_your_host_system_for_using_docker).  Follow the instructions for your specific platform.  After running the eval command above for docker, a local docker client will be able to use those variables to access the docker daemon running in your cdk VM.
