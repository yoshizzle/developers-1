---
# vim: tw=80
title: Creating a Linode
blurb: How to use the official python library.
layout: guide
---

With the release of api v2, Linode has also released an official python library.  This guide
will walk you through using the library to make single-user and multi-user applications
that take advantage of all of the features of the new Linode API.

## Getting Started

The official linode python library is open-source on [github](), and can be installed through
pypi with:

```pip install <package-name>```

Please note that this package only supports
python3, so on some systems you may need to run `pip3 install <package-name>` use the correct version
of python.

## OAuth Workflow

The official Linode python library supports both single-user and multi-user applications, backed
by Linode's OAuth server.  If you are reading this guide to develop a single-user application
(something that knows uses only one token to access the API and only manages one Linode account)
you can safely skip this section.

When a users comes to your application and wants to give it access to their Linode account, you
will direct the user through an OAuth login process, and in the end receive an OAuth token that
gives you some degree of access to the user's Linode account.  In order for them to grant you 
access, [read through this section of the API guide](/reference/#authentication).

To interface with the OAuth service in python, we will use the `LinodeLoginClient`.  Here is an
example of a simple implementation:

{% highlight python %}
from linode import LinodeLoginClient, OAuthScopes

client_id = 'my-client-id'
client_secret = 'my-client-secret'

def get_login_client(): 
    return LinodeLoginClient(client_id, client_secret, base_url='https://alpha.login.linode.com')

def get_login_url():
    return get_login_client().generate_login_url(scopes=OAuthScopes.Linodes.view)

def complete_login(code):
    token, scopes = get_login_client().finish_oauth(code)

    if not OAuthScopes.Linodes.view in scopes:
        raise ValueError('Insufficient scopes granted by user!')
    return token
{% endhighlight %}

In this example, we create a `LinodeLoginClient` with our client ID and client secret, and set the
base_url to alpha (base_url is optional, and defaults to pointing at the real deployment - alpha
is a great place to test your code). We then use the client to generate a login URL for the end 
user with the OAuth scopes our application requires.  The user should be redirected to this URL to 
complete the OAuth flow.  Once they're done, our application should receive the callback response,
which will have the user's temporary code in the query string.  We take this code and again ask
the client to finish the OAuth flow using this code, and are given the OAuth scopes the user granted
us, as well as the new OAuth token we can use to access the user's account.  It is always sane to
check that at least the minimum required OAuth scopes were granted to our application, and to 
display an error to the user and ask them to reauthenticate with the required scopes.

## Single User Applications

If your application is only managing once Linode account (your own), you can create an OAuth token
through the [login site]() to use in your application.  It can be used in the same way as any other
token.

## Connecting to the API

Once you have an OAuth token, connecting to the API is as simple as creating a `LinodeClient`.  This
client will handle all communications to the API for a given user.

{% highlight python %}
from linode import LinodeClient

oauth_token = 'my-token'

client = LinodeClient(oauth_token, base_url='https://alpha.api.linode.com/v2')
{% endhighlight %}

## Objects

The Linode pyton library is completely object-oriented, and every API resource has an object that
represents it.  If you know the ID of an object, it can be created with a `LinodeClient` and ID and
used as you please:

{% highlight python %}
from linode import Linode

linode = Linode(client, 'lnde_123')
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
from linode import Disk

disk = Disk(client, 'disk_123', 'lnde_123')
{% endhighlight %}

## Lists of Objects

If you don't know an object's ID, that's fine - we can find it in a list.  All root API endpoints
(`/linodes`, `/zones`, `/datacenters`, etc) can be accessed as a list from the LinodeClient object,

{% highlight python %}
# list datacenters
datacenters = client.get_datacenters()

# list linodes on this account
linodes = client.get_linodes()
{% endhighlight %}

#### Filtering

All listing methods support filtering - pass in the attributes you want to filter the list on, and
only objects with matching values will be included in the resulting list.  Please note that a list is
always returned, so if you are expecting a single object you need to get the first object in the list.

{% highlight python %}
# get all linodes in newark
newark = client.get_datacenters(lable='newark')[0]
newark_linodes = client.get_linodes(datacenter=newark)
{% endhighlight %}

## Examples

There are two official exmaple projects using the linode python library, found here:

[**Single User Application**]() - an application that allows users to sign up and receive a single
linode running your software - a simple reseller prototype.

[**Multi-User Application**]() - an application that creates a new linode running your software for a
user through an "Install on Linode" button.
