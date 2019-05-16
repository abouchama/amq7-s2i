# amq7-s2i

##S2I Build OpenShift:

Create first a new project (namespace) called Broker:

oc new-project broker

Now let's s2i our conf to the jboss-amq image

```
$ oc new-build registry.redhat.io/amq-broker-7/amq-broker-73-openshift:7.3~https://github.com/abouchama/amq7-s2i.git
--> Found Docker image 213daa8 (2 weeks old) from registry.redhat.io for "registry.redhat.io/amq-broker-7/amq-broker-73-openshift:7.3"

    Red Hat AMQ Broker 7.3.0 
    ------------------------ 
    A reliable messaging platform that supports standard messaging paradigms for a real-time enterprise.

    Tags: messaging, amq, java, jboss, xpaas

    * An image stream tag will be created as "amq-broker-73-openshift:7.3" that will track the source image
    * A source build using source code from https://github.com/abouchama/amq7-s2i.git will be created
      * The resulting image will be pushed to image stream tag "amq7-s2i:latest"
      * Every time "amq-broker-73-openshift:7.3" changes a new build will be triggered

--> Creating resources with label build=amq7-s2i ...
    imagestream.image.openshift.io "amq7-s2i" created
    buildconfig.build.openshift.io "amq7-s2i" created
--> Success
```

To stream the build progress, run 'oc logs -f bc/amq63-basic-s2i', You can see here that our conf openshift-activemq.xml has been copied to the image stream:

```
$ oc logs -f bc/amq7-s2i
Cloning "https://github.com/abouchama/amq7-s2i.git" ...
	Commit:	e96946f128fcf1e9072c99bf631b7104f758c01d (Update broker.xml)
	Author:	Abdellatif BOUCHAMA <abdellatif.bouchama@gmail.com>
	Date:	Thu May 16 16:49:07 2019 +0200
Using registry.redhat.io/amq-broker-7/amq-broker-73-openshift@sha256:5003e612962f130ff2e3e12fae6b0a2d808c2ef65ec97300ed3060db72481067 as the s2i builder image
INFO Copying configuration from configuration to /opt/amq/conf...
broker.xml
Pushing image 172.30.1.1:5000/broker/amq7-s2i:latest ...
Pushed 2/5 layers, 40% complete
Pushed 3/5 layers, 73% complete
Pushed 4/5 layers, 99% complete
Pushed 5/5 layers, 100% complete
Push successful
```
