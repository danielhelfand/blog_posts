# Using a custom builder image on Red Hat OpenShift with OpenShift Do

**Published here:** https://developers.redhat.com/blog/2019/07/15/using-a-custom-builder-image-on-red-hat-openshift-with-openshift-do/

One of the things I enjoy most about using [Red Hat OpenShift](https://developers.redhat.com/openshift/) is the Developer Catalog. The Developer Catalog is a central location where a team working with Red Hat OpenShift can encapsulate and share how application components and services are deployed.

The Developer Catalog is often used to define an infrastructure pattern referred to as a builder image. A builder image is a container image that supports a particular language or framework, following [best practices and Source-to-Image (s2i) specifications](https://docs.openshift.com/container-platform/4.1/openshift_images/create-images.html?extIdCarryOver=true&sc_cid=701f2000001OH7EAAW).

The OpenShift Developer Catalog provides several standard builder images supporting applications written in Node.js, Ruby, Python, and more. And while the Developer Catalog has many easy ways to get started deploying several supported languages, the catalog is also flexible in allowing you to add your own builder images to support an infrastructure pattern that is not preloaded in the catalog.

## Custom Builder Images

One language that is currently not preloaded into the catalog is Golang. To add a Golang builder image, an imagestream can be used to define what container image to use to support a Golang component. An imagestream can be written in YAML or JSON to define properties of the builder image, including where to pull the image definition from and descriptions of the image that will display in the Developer Catalog web console.

I used the imagestream found [here](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream) to add Golang to a Red Hat OpenShift 4 Developer Catalog. The first property we are interested in defining as part of an imagestream is the [kind property](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L3), which is used to specify that an imagestream is being created. We can also define a [display name](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L6) that will be available in the catalog through the web console.

The main [tags property](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L11) of the imagestream definition can be used to define different versions of a builder image that are supported. Additionally, each version defined within the tags property can have its own tags where it can be specified the image is a builder image by including a [builder tag](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L21).

Using the from property, we can specify what image to use and where to pull the image from. In the [Golang example](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L44), the image is pulled from DockerHub and is a Docker image.

The final step in adding the Golang builder image is to log in to your Red Hat OpenShift cluster using oc login and then run the following:

```
curl -kL https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62/raw 2> /dev/null | oc apply -n openshift --as system:admin -f -
```

With the command above, a Golang option has been added under the OpenShift namespace and will now be available to support Golang applications. Now that this builder image is available in the catalog, how can it quickly be utilized to deploy an application component and test that the image is able to support that component?

Using [OpenShift Do (odo)](https://developers.redhat.com/blog/2019/05/03/announcing-odo-developer-focused-cli-for-red-hat-openshift/), the developer-focused CLI for OpenShift, we can easily leverage builder images available in the Developer Catalog. To see what options are available, odo features a command to list builder images (i.e., components) available in the catalog:

```
$ odo catalog list components

NAME               PROJECT      TAGS
dotnet             openshift    1.0,1.1,2.1,2.2,latest
golang             openshift    1.10.2,latest
httpd              openshift    2.4,latest
java               openshift    8,latest
modern-webapp      openshift    10.x,latest
nginx              openshift    1.10,1.12,1.8,latest
nodejs             openshift    10,6,8,8-RHOAR,latest
perl               openshift    5.24,5.26,latest
php                openshift    7.0,7.1,7.2,latest
python             openshift    2.7,3.5,3.6,latest
ruby               openshift    2.3,2.4,2.5,latest
```

Using the OpenShift Golang example shown [here](https://github.com/sclorg/golang-ex), I can run the following to create a project in my OpenShift cluster, create a Golang component that will use the Golang builder image created via an imagestream, expose the component via a URL, and then deploy the component and have it running on OpenShift:

```
# Clone the golang example application and go into its root directory
$ git clone https://github.com/sclorg/golang-ex
$ cd golang-ex

# Create the configuration for the golang example application to be
# deployed to OpeShift using odo and deploy it to OpenShift

$ odo project create goproject
$ odo create golang golang-ex --port 8080
$ odo url create golang-ex --port 8080
$ odo push
```

When the deployment is complete, the Golang component can be accessed via the URL created by odo. Clicking on the URL should show the following greeting:

>Hello OpenShift!

Imagestreams make it easier to promote reuse of infrastructure patterns so development teams can focus on creating applications rather than how they are hosted. odo further extends their capabilities by providing a CLI that makes it simple to pick and choose what infrastructure to utilize when developing applications on OpenShift.

More information on imagestreams can be found in the [Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.1/openshift_images/images-understand.html#images-imagestream-use_images-understand?extIdCarryOver=true&sc_cid=701f2000001OH7EAAW). Official Red Hat and community-supported imagestreams can be found on the [openshift/library GitHub repository](https://github.com/openshift/library/tree/master/community) as well as under the [Software Collections GitHub organization](https://github.com/sclorg).

For information on odo, you can visit the [OpenShift Do](https://openshiftdo.org/) website and also check out the project’s [GitHub repository](https://github.com/openshift/odo).
