---
# vim: tw=80
title: Creating a Linode
blurb: Starting from nothing and ending with a running server.
layout: guide
---

## Selecting a datacenter

First you will want to select a datacenter to deploy your new Linode into.
The API provides an endpoint that allows you to do just that. To retrieve a
list of active datacenters, run the command below:

    curl https://api.linode.com/v2/datacenters

Note that since the datacenter list is public information, you don't need to
send your authorization token (although it will still work if you do). The
above command will return a JSON object like the following:

{% highlight json %}
{
    "datacenters": [
        {
            "label": "Dallas, TX",
            "id": "dctr_2"
        },
        {
            "label": "Fremont, CA",
            "id": "dctr_3"
        },
        {
            "label": "Atlanta, GA",
            "id": "dctr_4"
        },
        {
            "label": "Newark, NJ",
            "id": "dctr_6"
        },
        {
            "label": "London, UK",
            "id": "dctr_7"
        },
        {
            "label": "Singapore, SG",
            "id": "dctr_9"
        },
        {
            "label": "Tokyo, JP",
            "id": "dctr_8"
        }
    ],
    "total_pages": 1,
    "page": 1,
    "total_results": 7
}
{% endhighlight %}

The datacenter list is pretty self-explanatory: there are 7 available
datacenters, and their geographical locations are provided in the `label`
field. The `id` field is a unique ID which you'll use to refer to the
datacenter you want to select. For this example, we'll go with the
"Newark, NJ" datacenter with ID "dctr_6".

## Selecting a service plan

Once you have a datacenter in mind, the next step is to choose a Linode
service plan. Run the following curl command to retrieve a list of available
Linode sizes:

    curl https://api.linode.com/v2/services/linode

The above command will return a JSON object like the following:

{% highlight json %}
{
    "total_pages": 1,
    "total_results": 1,
    "page": 1,
    "services": [
        {
            "label": "Linode 1024",
            "vcpus": 1,
            "mbits_out": 125,
            "disk": 24,
            "hourly_price": 1,
            "service_type": "linode",
            "ram": 1024,
            "id": "serv_112",
            "monthly_price": 1000,
            "transfer": 2000
        }/*, etc */
    ]
}
{% endhighlight %}

Each service represents an available Linode service plan. You can review
the details of each plan, such as the number of Virtual CPUs, memory, disk
space, monthly price, etc. For explanations of each field, see the
[complete service object reference](/reference/#object-service). Once you have
selected the service plan you want to launch, note its ID.

## Selecting a distribution

Now you need to choose a Linux distribution to deploy to your new Linode. Just
like selecting a service and a datacenter, issue a call to the API, this time
for a list of available distributions:

    curl https://api.linode.com/v2/distributions

This will provide you with a list of distributions:

{% highlight json %}
{
    "total_pages": 1,
    "total_results": 1,
    "page": 1,
    "distributions": [
        {
            "vendor": "Debian",
            "created": "2014-09-24T13:59:32",
            "minimum_image_size": 600,
            "id": "dist_130",
            "label": "Debian 7",
            "recommended": true,
            "experimental": false,
            "x64": true
        }
    ]
}
{% endhighlight %}

For detailed information about each field, see the
[complete distribution reference](/reference/#object-distribution).
