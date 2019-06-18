# Adding a Custom Builder Image to an OpenShift Service Catalog

One of the things that I have enjoyed using most about OpenShift is a feature
that I think is commonly overlooked: the Developer Catalog. The Developer Catalog
is a centralized location where a development team working with OpenShift can
encapsulate and share how application components and services are deployed to an
OpenShift project.

A very common use case of the Developer Catalog is to define an infrastructure
pattern commonly referred to as a `builder image`. A `builder image` is a container
image used to support a particular language or framework.

The OpenShift Developer Catalog comes preloaded with `builder images` that can
support `Node.js`, `Ruby`, `Python`, `Red Hat Open JDK 8`, and more. And while the
Developer Catalog has many easy ways to get started deploying a number of supported
languages, the catalog is also flexible in allowing you to add your own `builder images`
to support an infrastructure pattern that is not preloaded in the catalog.

Recently, I was working with `Minishift`, which is a tool for running OpenShift
3.11 and below locally. I was using OpenShift Do (`odo`), the developer-focused CLI
for OpenShift, to try and deploy a Java component as part of an application. `odo`
uses the Developer Catalog to deploy local source code to OpenShift and start up
that source code on a container that will support the local language used with a
couple of CLI commands.

This unfortunately didn't work because the `Minishift` Developer Catalog doesn't come
with a Java option. So I started learning about how to add a custom `builder image`
in the event the Developer Catalog doesn't support a particular language or framework.

The method I used to add a Java builder image with `Red Hat Open JDK 8` to the
Developer Catalog was an `image stream` to define what image I would use
to support my Java component. An `image stream` can be defined in YAML or JSON to
define properties of the `builder image`, including where to pull the image definition
from and descriptions that will display in the Developer Catalog web console.

The `image stream` I used to add Java to the `Minishift` catalog can be found
[here](https://gist.github.com/danielhelfand/ad53ceb06cd7619790a1700a95e3c93a).
The first property we are interested in defining as part of an image stream is the
`kind` property, which is used to specify that an `image stream` being created as
shown [here](https://gist.github.com/danielhelfand/ad53ceb06cd7619790a1700a95e3c93a#file-red-hat-openjdk-8-imagestream-L3).
We can also define a display name that will be available in the catalog UI through
the web console as shown [here](https://gist.github.com/danielhelfand/ad53ceb06cd7619790a1700a95e3c93a#file-red-hat-openjdk-8-imagestream-L9).

The `tags` property of the `image stream` definition is used to define different versions
of a `builder image` that are supported (e.g., using OpenJDK 8-1.5 or 8-1.6). Using the
`from` property, we can define what image to use and where to pull the image from. In
the [java example](https://gist.github.com/danielhelfand/ad53ceb06cd7619790a1700a95e3c93a#file-red-hat-openjdk-8-imagestream-L28),
the image is pulled from the Red Hat Container Registry and is a `Docker` image.

The final step in adding the Java `builder image` is to login to your OpenShift cluster
using `oc login` and run the following:

```
curl -kL https://gist.github.com/danielhelfand/ad53ceb06cd7619790a1700a95e3c93a/raw 2> /dev/null | oc apply -n openshift --as system:admin -f -
```

With the command above, a Java option has been added under the openshift namespace
and will now be available. The above command can be used to add `Red Hat OpenJDK8`
to `Minishift` or any OpenShift cluster assuming proper permissions are available to
the user adding the `builder image`.

So we have seen how to add a Java option to the Developer Catalog, but how can these
`builder images` be utilized to deploy an application component? Using `odo`, we can easily
leverage `builder images` available in the Developer catalog. To see what option are
available, `odo` features a command to list components available in the catalog:

```
$ odo catalog list components

NAME              PROJECT       TAGS
dotnet            openshift     1.0,1.1,2.1,2.2,latest
httpd             openshift     2.4,latest
java              openshift     8,latest
modern-webapp     openshift     10.x,latest
nginx             openshift     1.10,1.12,1.8,latest
nodejs            openshift     10,6,8,8-RHOAR,latest
perl              openshift     5.24,5.26,latest
php               openshift     7.0,7.1,7.2,latest
python            openshift     2.7,3.5,3.6,latest
ruby              openshift     2.3,2.4,2.5,latest
```

In OpenShift 4, the Developer Catalog features a Java component as shown above. To
show the simplicity of adding components to catalog, I am going to add a `golang`
component to the Developer catalog using the command below:

```
curl -kL https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62/raw 2> /dev/null | oc apply -n openshift --as system:admin -f -
```

I am now presented with a `golang` component in my Developer Catalog that I can
use to deploy a `golang` application using `odo`:

```
NAME              PROJECT       TAGS
dotnet            openshift     1.0,1.1,2.1,2.2,latest
golang            openshift     1.10.2,latest
httpd             openshift     2.4,latest
java              openshift     8,latest
modern-webapp     openshift     10.x,latest
nginx             openshift     1.10,1.12,1.8,latest
nodejs            openshift     10,6,8,8-RHOAR,latest
perl              openshift     5.24,5.26,latest
php               openshift     7.0,7.1,7.2,latest
python            openshift     2.7,3.5,3.6,latest
ruby              openshift     2.3,2.4,2.5,latest
```

Using the OpenShift golang example shown [here](https://github.com/sclorg/golang-ex),
I can run the following to create a project in my OpenShift cluster, create a
golang component that will use the `golang` component created via an `image stream`,
expose the component via a url, and then deploy the component and have it running on
OpenShift:

```
odo project create goproject
odo create golang golang-ex --port 8080
odo url create golang-ex --port 8080
odo push
```

When the deployment is complete, the `golang` component can be accessed via the url
created with the following:

```
Hello OpenShift!
```
