# gateway-vertx

This microservice can only be run locally if the inventory and catalog services are also running locally

* Inventory github project: [https://github.com/bugbiteme/inventory-wildfly-swarm]()
* Catalog github project: [https://github.com/bugbiteme/catalog-spring-boot]()

The gateway microservice also cannot be run in OpenShift unless the inventory and catalog services are also running in OpenShift.

Instructions for deploying the entire coolstore application will be available soon.

## Run gateway-vertx as a stand-alone RestAPI service running locally

Since the API Gateway requires the Catalog and Inventory services to be running, start all three services simultaneously and verify that the API Gateway works as expected.

Open a new terminal window and start the Catalog service:

~~~~
$ cd catalog-spring-boot
$ mvn spring-boot:run
~~~~

Open another new terminal window and start the Inventory service:

~~~~
$ cd inventory-wildfly-swarm
$ mvn wildfly-swarm:run
~~~~

Now that Catalog and Inventory services are up and running, start the API Gateway service in a new terminal window:

~~~~
$ cd gateway-vertx
$ mvn vertx:run
~~~~

*You will see the following exception in the logs:* `java.io.FileNotFoundException: /.../kubernetes.io/serviceaccount/token`

*This is expected and is the result of Vert.x trying to import services form OpenShift. Since you are running the API Gateway on your local machine, the lookup fails and falls back to the local service lookup. It’s all good!*


validate it is running using curl (or a web browser)

`curl http://localhost:9001/8080/api/products`
 
 output

~~~~
[ {
  "itemId" : "329299",
  "name" : "Red Fedora",
  "desc" : "Official Red Hat Fedora",
  "price" : 34.99,
  "availability" : {
    "quantity" : 35
  }
},
...
]
~~~~

terminate service `ctrl-c` in all open terminals

## Build and deploy gateway service on OpenShift. 

Assuming you have logged into OpenShift, make sure you are in the coolstore project:

`$ oc project coolstore`

Launching the gateway requires an additional step, so that it can talk to the other services that it depends on. 

Vert.x service discovery integrates into OpenShift service discovery via OpenShift REST API and imports available services to make them available to the Vert.x application. Security in OpenShift comes first and therefore accessing the OpenShift REST API requires the user or the system (Vert.x in this case) to have sufficient permissions to do so. All containers in OpenShift run with a serviceaccount (by default, the project `default` service account) which can be used to grant permissions for operations, for instance, accessing the OpenShift REST API. 

Grant permission to the API Gateway to be able to access OpenShift REST API and discover services.

`$ oc policy add-role-to-user view -n coolstore -z default`

To build and deploy the gateway service into OpenShift using the fabric8 maven plugin, run the following Maven command:

`$ mvn fabric8:deploy`

While you are waiting for the deploy command to complete, you can log into the OpenShift web consol and check the progress of your deployment, and even view the build and deployment logs, which should look very similar to the messages seen when running the service locally.

Once the service ha been deployed, you can get the url by running

`$ oc get route`

validate it is running using curl (or a web browser)

`curl http://SERVICE_ROUTE_OPENSHIFT_URL/api/products`
 
 output
 
~~~~
[ {
  "itemId" : "329299",
  "name" : "Red Fedora",
  "desc" : "Official Red Hat Fedora",
  "price" : 34.99,
  "availability" : {
    "quantity" : 35
  }
},
...
]
~~~~

Original source:
[http://guides-cdk-roadshow.b9ad.pro-us-east-1.openshiftapps.com/index.html#/workshop/roadshow/module/vertx]()


