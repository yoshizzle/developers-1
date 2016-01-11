---
# vim: tw=80
title: Creating a Linode
blurb: How to use the official python library.
layout: guide
---

With the release of API 2, Linode has also released an official python library.  This guide
will walk you through using the library to make single-user and multi-user applications
that take advantage of all of the features of the new Linode API.

## Getting Started

The official linode python library is open-source on [github](), and can be installed through
pypi with:

```pip install <package-name>```

Please note that this package only supports
python3, so on some systems you may need to run `pip3 install <package-name>` use the correct version
of python.

## Getting an OAuth Token

When working on an application that only deals with your own lindoes, acquiring an OAuth token is
easy.  Just log in to https://login.alpha.linode.com, click 'Manage Applications and Tokens', and
then click 'Generate New Token' and copy the token displayed on the screen.

#### Generating OAuth Tokens

If you need to generate an OAuth token using a client ID and client secret, as you would in an
application where you manage linodes for a user, use the LinodeLoginClient.  This requires
you to have set up an OAuth application at {{ site.login_root }}.

{% highlight python %}
>>> from linode import LinodeLoginClient, OAuthScopes
>>> login_client = LinodeLoginClient('my-client-id', 'my-client-secret', base_url='https://login.alpha.linode.com')
>>> login_client.generate_login_url(scopes=OAuthScopes.Linodes.view)
'https://login.alpha.linode.com/oauth/authorize?client_id=my-client-id&scopes=linodes%3Aview'
{% endhighlight %}

Visit this URL in a browser and complete the login process.  The page you are sent to when login
is successful will have a `code=` in the query string - get this value to continue.

{% highlight python %}
>>> token, scopes = login_client.finish_oauth('code-from-query-string')
{% endhighlight %}

In a real-world scenario, your application would redirect a users through to the login service, then
receive the callback once login was complete and capture the code from the query string.  For a
more practical example, see the [multi-user example application]().

## Connecting to the API

Once you have an OAuth token, connecting to the API is as simple as creating a `LinodeClient`.  This
client will handle all communications to the API for a given user.

{% highlight python %}
>>> from linode import LinodeClient
>>> client = LinodeClient('my-token', base_url='https://api.alpha.linode.com/v2')
{% endhighlight %}

## Objects

The Linode pyton library is completely object-oriented, and every API resource has an object that
represents it.  If you know the ID of an object, it can be created with a `LinodeClient` and ID and
used as you please:

{% highlight python %}
>>> from linode import Linode
>>> linode = Linode(client, 'lnde_123')
{% endhighlight %}

Each object has attributes that match those documented in [the api spec](/reference/#objects), and only those
marked 'editabled' may be changed.  All objects share the following:

| id | The id of this object in the Linode API |
| save | This function calls PUT on this object, saving any changes to mutable fields |
| invalidate | This function expires all attributes of the object except identifiers so they will be refreshed |
| delete | This function calls DELETE on this objects.  Use with caution |

Objects are lazy-loaded, so creating the object will not reach out to the API until a field it doesn't
have is referneced.

Objects may also have lists of related objects - if an object's endpoing can be extended after the ID, the
extended path is an attribute of that object which returns the related list of derived objects.

For example, the `/linodes/:id/disks` implies that the Linode object hs a disks attribute - which it does.
Refernecing this attribute will return the result of this API call - in this case, a list of Disks.

#### Derived Objects

Some objects belong to other objects - for instance, a Linode has Disks, so a Disk is a derived object.
If the endpoint for a resource includes another resource's ID, that resource is derived from the resource
whose ID preceeds it.  In these cases, we must provide the parent object's ID when creating a new instance
of the derived object.

{% highlight python %}
>>> from linode import Disk
>>> disk = Disk(client, 'disk_123', 'lnde_123')
{% endhighlight %}

## Lists of Objects

If you don't know an object's ID, that's fine - we can find it in a list.  All root API endpoints
(`/linodes`, `/zones`, `/datacenters`, etc) can be accessed as a list from the LinodeClient object,

{% highlight python %}
>>> datacenters = client.get_datacenters()
>>> linodes = client.get_linodes()
{% endhighlight %}

#### Filtering

All listing methods support filtering - pass in the attributes you want to filter the list on, and
only objects with matching values will be included in the resulting list.  Please note that a list is
always returned, so if you are expecting a single object you need to get the first object in the list.

{% highlight python %}
# get all linodes in newark
>>> newark = client.get_datacenters(lable='newark')[0]
>>> newark_linodes = client.get_linodes(datacenter=newark)
{% endhighlight %}

## Examples

There are two official exmaple projects using the linode python library, found here:

[**Single User Application**]() - an application that allows users to sign up and receive a single
linode running your software - a simple reseller prototype.

[**Multi-User Application**]() - an application that creates a new linode running your software for a
user through an "Install on Linode" button.
