# Guide to Installing the Storefront Demo on OpenShift

## Pre-requisites

- OpenShift cluster provisioned
- Java 11 on your local machine
- OpenShift oc cli installed on your local machine

## OpenShift Installation



1. Clone all the repos in the `https://github.com/storefront-demo`

    You should have the following directories on your system:

    ```
    .
    ├── catalog-ms-quarkus
    ├── customer-ms-quarkus
    ├── documentation
    ├── inventory-ms-quarkus
    ├── orders-ms-quarkus
    ├── storefront-databases
    └── storefront-ui
    ```
2. Create the storefront app database

    In the `storefront-databases` directory run the following command:

    `./setup_databases.sh storefront-dev`

    This will create an OpenShift project call `storefront-dev` and install the databases as well as populate them with the right data.

3. Install KeyCloak

    - Follow the approach found at this url:
    https://cloudnativereference.dev/related-repositories/keycloak/#start-keycloak-on-openshift

    - Test KeyCloak:  TODO

4. Deploy Inventory microservice

    In the `inventory-ms-quarkus` directory run the following command:

    `./mvnw clean package -Dquarkus.kubernetes.deploy=true`

    Test the inventory service by running these commands:
    ```
    // port forward to the inventory service
    oc port-forward service/inventory-ms-quarkus 80:8080

    // check the health endpoint
    curl http://localhost:8080/health

    // invoke the inventory api
    curl http://localhost:8080/micro/inventory
    ```
5. Deploy Catalog microservice

    In the `catalog-ms-quarkus` directory run the following command:

    `./mvnw clean package -Dquarkus.kubernetes.deploy=true`


    - Now create the route as follows.

    ```
    oc expose svc catalog-ms-quarkus
    ```

    - Grab the route.

    ```
    oc get route catalog-ms-quarkus --template='{{.spec.host}}'
    ```

    Test the catalog service by running these commands:
    ```
    curl http://<your-route>/micro/items
    ```