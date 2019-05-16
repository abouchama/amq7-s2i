# How to deploy AMQ7 on OpenShift using S2I

Configuration of the AMQ7 image can also be modified using the S2I (Source-to-image) feature.

Custom AMQ broker configuration can be specified by creating an broker.xml file inside the git directory of your application’s Git project root. On each commit, the file will be copied to the conf directory in the AMQ root and its contents used to configure the broker.

Create first a new project (namespace) called Broker:
```
oc new-project broker
```
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

To stream the build progress, run 'oc logs -f bc/amq7-s2i', You can see here that our conf broker.xml has been copied to the image stream:

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

Create the service account "amq-service-account"
```
echo '{"kind": "ServiceAccount", "apiVersion": "v1", "metadata": {"name": "amq-service-account"}}' | oc create -f -
serviceaccount "amq-service-account" created
```

Ensure the service account is added to the namespace for view permissions... (for pod scaling)
```
oc policy add-role-to-user view system:serviceaccount:broker:amq-service-account
```

Install templates in the namespace broker:
```
for template in amq-broker-73-basic.yaml \
amq-broker-73-ssl.yaml \
amq-broker-73-custom.yaml \
amq-broker-73-persistence.yaml \
amq-broker-73-persistence-ssl.yaml \
amq-broker-73-persistence-clustered.yaml \
amq-broker-73-persistence-clustered-ssl.yaml;
 do
 oc replace --force -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/73-7.3.0.GA/templates/${template}
 done
```

Let's get now, our image stream URL:

```
$ oc get is
NAME                      DOCKER REPO                                      TAGS         UPDATED
amq-broker-73-openshift   172.30.1.1:5000/broker/amq-broker-73-openshift   7.3,latest   33 minutes ago
amq7-s2i                  172.30.1.1:5000/broker/amq7-s2i                  latest       About a minute ago
```

Specify the image (172.30.1.1:5000/broker/amq7-s2i) in one of the templates installed above in the parameter 'IMAGE':
For instance:

```
oc process amq-broker-73-basic -p APPLICATION_NAME=broker-s2i -p AMQ_NAME=broker-s2i -p AMQ_USER=user -p AMQ_PASSWORD=password -p AMQ_PROTOCOL=openwire,amqp,stomp,mqtt,hornetq -p IMAGE_STREAM_NAMESPACE=broker -p IMAGE=172.30.1.1:5000/broker/amq7-s2i -n broker | oc create -f -
```

Update of broker.xml

You should setup the GitHub webhook URL in order to trigger a new build after each update.

You can also do it manually, like following:
```
$ git commit -am "changing my broker conf"

$ git push -u origin master

$ oc start-build amq7-s2i -n broker
build "amq7-s2i-2" started

$ oc deploy broker-amq --latest -n broker
Started deployment #2
Use 'oc logs -f dc/broker-amq' to track its progress.
```
