---
# vim: tw=80
title: Python Guide 
blurb: How to use the official python library, linode-api
layout: guide
---

<style>
table {
    table-layout: fixed;
}
</style>

With the release of API 4, Linode has also released an official python library.  This guide
will walk you through using the library to make single-user and multi-user applications
that take advantage of all of the features of the new Linode API.

## Getting Started

The official Linode python library is open-source on [github](https://github.com/Linode/python-api),
and can be installed through pypi with:

```pip install linode-api```

## Getting an OAuth Token

When working on an application that only deals with your own Linodes, acquiring an OAuth token is
easy.  Just log in to https://login.alpha.linode.com, click 'Manage Applications and Tokens', and
then click 'Generate New Token' and copy the token displayed on the screen.

#### Generating OAuth Tokens

If you need to generate an OAuth token using a client ID and client secret, as you would in an
application where you manage Linodes for a user, use the LinodeLoginClient.  This requires
you to have set up an OAuth application at {{ site.login_root }}.

{% highlight python %}
>>> from linode import LinodeLoginClient, OAuthScopes
>>> login_client = LinodeLoginClient('my-client-id', 'my-client-secret', base_url='https://login.alpha.linode.com')
>>> login_client.generate_login_url(scopes=OAuthScopes.all)
'https://login.alpha.linode.com/oauth/authorize?client_id=my-client-id&scopes=%2A'
{% endhighlight %}

Visit this URL in a browser and complete the login process.  The page you are sent to when login
is successful will have a `code=` in the query string - get this value to continue.

{% highlight python %}
>>> token, scopes = login_client.finish_oauth('code-from-query-string')
{% endhighlight %}

In a real-world scenario, your application would redirect users through to the login service, then
receive the callback once login was complete and capture the code from the query string.  For a
more practical example, see the [multi-user example application]().

## Connecting to the API

Once you have an OAuth token, connecting to the API is as simple as creating a `LinodeClient`.  This
client will handle all communications to the API for a given user.

{% highlight python %}
>>> from linode import LinodeClient
>>> client = LinodeClient('my-token', base_url='https://api.alpha.linode.com/v4')
{% endhighlight %}

## Objects

The Linode python library is completely object-oriented, and every API resource has an object that
represents it.  If you know the ID of an object, it can be created with a `LinodeClient` and ID and
used as you please:

{% highlight python %}
>>> from linode import Linode
>>> linode = Linode(client, 'lnde_123')
{% endhighlight %}

Each object has attributes that match those documented in [the API spec](/reference/#objects), and only those
marked 'editable' may be changed.  All objects share the following:

| id | The id of this object in the Linode API |
| save | This function calls PUT on this object, saving any changes to mutable fields |
| invalidate | This function expires all attributes of the object except identifiers so they will be refreshed |
| delete | This function calls DELETE on this object.  Use with caution |

Objects are lazy-loaded, so creating the object will not reach out to the API until a field it doesn't
have is referenced.

#### Derived Objects

Some objects belong to other objects - for instance, a Linode has Disks, so a Disk is a derived object.
If the endpoint for a resource includes another resource's ID, that resource is derived from the resource
whose ID preceeds it.  In these cases, we must provide the parent object's ID when creating a new instance
of the derived object.

{% highlight python %}
>>> from linode import Disk
>>> disk = Disk(client, 'disk_123', 'lnde_123')
{% endhighlight %}

A list of derived objects may also be accessed through their parent object as an attribute,
named based on the extension to the parent object's URL (i.e. `/linodes/lnde_123/disks` means
a Linode object will have a disks attribute):

{% highlight python %}
>>> linode = Linode(client, 'lnde_123')
>>> disks = linode.disks
{% endhighlight %}

## Lists of Objects

If you don't know an object's ID, that's fine - we can find it in a list.  All root API endpoints
(`/linodes`, `/zones`, `/datacenters`, etc) can be accessed as a list from the LinodeClient object:

{% highlight python %}
>>> datacenters = client.get_datacenters()
>>> linodes = client.get_linodes()
{% endhighlight %}

#### Filtering

All list methods support filtering in a SQLAlchemy-like syntax.  All API object classes have
attributes for all properties listed as filterable in the object reference.  You can search
endpoints like so:

{% highlight python %}
# get all linodes in newark
>>> from linode import Datacenter, Linode
>>> newark = client.get_datacenters(Datacenter.label=='Newark')[0]
>>> newark_linodes = client.get_linodes(Linode.datacenter==newark)
>>> foo_linodes = client.get_linodes(Linode.label.contains('foo'), Linode.datacenter == 'dctr_1')
{% endhighlight %}

Multiple filters are combined with an "and" operator.  You can also use "or" (either explicitly or
implicitly):

{% highlight python %}
>>> foobar_linodes = client.get_linodes((Linode.label.contains('foo')) | (Linode.label.contains('bar')))
>>> from linode import or_
>>> foorbaz_linodes = client.get_linodes(or_(Linode.label.contains('foo'), Linode.label.contains('baz')))
{% endhighlight %}

In addition to `==` and `contains`, objects can be filtered with the `>`, `<`, `>=`, and `<=` operators.

## Creating Resources

Through the LinodeClient you can create resources, with parameters matching the resource's POST endpoint.
In the [API Reference](/reference/#ep-linodes) we see that to create a Linode, you need to provide a Service,
a Datacenter, and optionally, a source.  Here is an example of how one may do that:

{% highlight python %}
>>> from linode import Service, Datacenter, Distribution
>>> serv = client.get_services(Service.label == 'Linode 1024')[0]
>>> dc = client.get_datacenters(Datacenter.label.contains('Newark'))[0]
>>> distro = client.get_distributions(Distribution.vendor == 'Ubuntu')[0]
>>> l, root_pw = client.create_linode(serv, dc, source=distro)
{% endhighlight %}

In this example, we queried the API for the Service, Datacenter and Distribution we wanted, then
sent a request to create a Linode with those arguments.  In this case, the create function also
helpfully generated a root password for the Linode, which was returned alongside it (you can provide
a root password yourself with the `root_pass` argument, as per the [API Reference](/reference/#ep-linodes),
and this function will only return the created Linode).

In general, required parameters to a POST request are positional arguments of the LinodeClient's create
methods, and optional arguments are passed as keyword arguments with the same names.  Here are the
required argument lists:

#### Linodes

| service | a Service object |
| datacenter | a Datacenter object|

#### StackScripts

| label | a label for the StackScript |
| script | the stackscript body |
| distros | a list of Distribution objects this StackScript runs on |

#### Zones

| zone | the zone |
| master | if this zone is master - defaults to True |

### Creating Derived Objects

Parent objects have the ability to create derived resources.  This is done
in the same way as creating top-level objects, except that you use the parent object
in place of the LinodeClient.

#### Linodes

Creating configs

| kernel | a Kernel object |

Creating disks

| size | the size of the new disk |

#### Zones

Creating zone records

| record_type | the type of zone record to create |

## Examples

There are two official example projects using the linode python library, found here:

[**Single User Application**]() - an application that allows users to sign up and receive a single
Linode running your software - a simple reseller prototype.

[**Multi-User Application**]() - an application that creates a new Linode running your software for a
user through an "Install on Linode" button.
