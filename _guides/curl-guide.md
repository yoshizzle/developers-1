---
# vim: tw=80
title: Testing with curl
blurb: A crash course on curl and how to use it to try out the API.
layout: guide
---

[curl](http://curl.haxx.se/) is a simple and popular command line tool that
allows you to perform various kinds of HTTP requests. It may already be
installed on your system - run `curl --version` in your shell to check. Once
you've confirmed that you have it installed, we can use it to test out the
Linode API from the comfort of your shell.

## Unauthenticated Requests

You can perform anonymous HTTP requests against various resources on the API.
You can tell which ones are anonymous from the <span class="text-muted">
<i class="fa fa-lock"></i> Authenticated</span> indicator in the [reference
documentation](/reference). For API endpoints where this indicator is missing,
you're able to use curl to test them without any additional steps. For example,
we could [list supported distributions](/reference/#ep-distributions):

    curl https://{{ site.api_root }}/{{ site.api_version }}/distributions

This will give you a response like this:


{% highlight json %}
{
    "distributions": [
        {
            "created": "2014-10-24T15:48:04",
            "experimental": false,
            "id": "dist_133",
            "label": "Ubuntu 14.10",
            "minimum_image_size": 650,
            "recommended": true,
            "vendor": "Ubuntu",
            "x64": true
        }, # and so on
    ],
    "page": 1,
    "total_pages": 2,
    "total_results": 34
}
{% endhighlight %}

## Authenticated Requests

For many requests, you will have to authenticate as a particular user. To do so,
you need to create an OAuth client and log in as yourself with that client. A
full run-down of how authentication works is provided on the
[reference page](/reference#authentication), but we'll give you enough
information to get curl working here.

You can register your OAuth client [here](https://{{ site.login_root }}/apps). For
testing purposes, you can set your redirect URI to https://linode.com. Once
you've done so, you'll receive a **client ID** and a **client secret**. Keep the
client secret somewhere safe - do not share it. You can, however, share the
client ID, and you should use it now to log into your Linode account:

[https://{{ site.login_root }}/oauth/authorize?scopes=\*&client_id=**your_client_id**](https://{{ site.login_root }}/oauth/authorize?scopes=*&client_id=your_client_id)

Make sure to add your client ID to this URL before you press enter. Log into
your Linode account and you'll be redirected to a URL like this:

https://linode.com?code=**somecode**&...

Once you have **somecode**, you can exchange this code for an OAuth token with
curl:

    curl https://{{ site.login_root }}/oauth/token \
        -F client_id=[client ID] \
        -F client_secret=[client secret] \
        -F code=[somecode]

You'll have to fill in your client ID and client secret (from the client
registration earlier). You also want to put in the code you got a moment ago.
When you submit this, you'll get a response like this:

{% highlight json %}
{
    "access_token": "b37ab5503b438bcd671bc1e1f4caecb47ee8585cb34668ea8f483d00702078a1",
    "scopes": "*"
}
{% endhighlight %}

Now you have an access token! That was a bit of work, but you only have to do it
once. Write down your access token somewhere.

### Authentication Header

Now you can make requests with curl using your access token by adding `-H
"Authorization: token that_token"`. The <span class="text-muted"><i class="fa
fa-lock"></i> Authenticated</span> requests on the [reference page](/reference)
include this header in the curl examples. Try this, for example:

    curl -H "Authorization: token that_token" \ 
        https://{{ site.api_root }}/{{ site.api_version }}/linodes

This will give you a response like this:


{% highlight json %}
{
    "linodes": [
        {
            "alerts": {
                "cpu": {
                    "enabled": true,
                    "threshold": 90
                },
                "io": {
                    "enabled": true,
                    "threshold": 5000
                },
                "transfer_in": {
                    "enabled": true,
                    "threshold": 5
                },
                "transfer_out": {
                    "enabled": true,
                    "threshold": 5
                },
                "transfer_quota": {
                    "enabled": true,
                    "threshold": 80
                }
            },
            "created": "2015-02-19T15:34:05",
            "datacenter": {
                "id": "dctr_6",
                "label": "Newark, NJ"
            },
            "distribution": null,
            "group": "",
            "id": "lnde_871212",
            "ip_addresses": {
                "private": {
                    "ipv4": [],
                    "link-local": "fe80::f03c:91ff:fe33:a2e4"
                },
                "public": {
                    "failover": [],
                    "ipv4": [],
                    "ipv6": "2600:3c03::f03c:91ff:fe33:a2e4"
                }
            },
            "label": "linode871212",
            "lish_command": "ssh -t dude2@lish-newark.linode.com linode871212",
            "ssh_command": "ssh root@",
            "status": "being_created",
            "total_transfer": 2000,
            "updated": "2015-02-19T16:10:45"
        } # and so on
    ],
    "page": 1,
    "total_pages": 14,
    "total_results": 130
}
{% endhighlight %}
