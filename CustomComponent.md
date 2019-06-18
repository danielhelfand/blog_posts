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
