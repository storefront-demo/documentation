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

4. Create Secret ibm-java-en

    Create secret `ibm-java-env` in the OpenShift project containing the microservices.
    ```
    oc create secret generic ibm-java-env --from-literal=KEYCLOAK_CLIENT_ID=bluecomputeweb --from-literal=KEYCLOAK_CLIENT_SECRET=<your-client-secret-from-keycloak> -n storefront-dev
    ```

    This secret is used by the `orders` and `catalog` microservices.

5. Deploy Inventory microservice

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
6. Deploy Catalog microservice

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

7. Deploy Orders microservice    

    In the `orders-ms-quarkus` directory run the following command:

    `./mvnw clean package -Dquarkus.kubernetes.deploy=true`

    - Now create the route as follows.

    ```
    oc expose svc orders-ms-quarkus
    ```

    - Test the service

    ```
    Todo
    ```

8. Deploy Customer microservice    

    In the `customer-ms-quarkus` directory run the following command:

    `./mvnw clean package -Dquarkus.kubernetes.deploy=true`

    - Now create the route as follows.

    ```
    oc expose svc orders-ms-quarkus
    ```

    - Test the service

    ```
    Todo
    ```
9. Storefront UI running on OpenShift interacting with microservices on OpenShift

    Go to directory `storefront-ui`

    Edit the file `config/production.json` and update the `client_secret` and the multipe `service_name` fields. The service_names will need to reference the routes for the microserices that you deployed earlier.

    Push the changes to your forked copy of the repo:

    Run the command to deploy to OpenShift:
    ```
    oc new-app --name storefront-ui https://github.com/<your-git-org>/storefront-ui#hp-quarkus-version
    ```

    Expose the route
    ```
    oc expose service storefront-ui
    ```

    In a browser, go to the route.

10. Storefront UI running locally interacting with microservices on OpenShift (Optional)

    Go to directory `storefront-ui`

    Edit the file `config/default.json` and update the `client_secret` and the multipe `service_name` fields. The service_names will need to reference the routes for the microserices that you deployed earlier.


    Command to build the docker image:
    ```
    appsody build
    ```

    Command to run the docker container:
    ```
    docker run --name web \
    -e NODE_ENV=development \
    -p 3000:3000 \
    -d dev.local/web
    ```

    In browser, goto: 
    ```
    http://localhost:3000
    ```

    You can login as user `foo` pw `bar`.