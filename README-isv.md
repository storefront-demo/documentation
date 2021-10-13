# Guide to Installing the Storefront Demo on OpenShift using IBM Security Verify as the IDP

This version of the storefront app is using IBM Security Verify as the identity provider and installs the app in the `storefront-isv` namespace.

## Pre-requisites
- OpenShift cluster provisioned
- IBM Security Verify tenant provisioned (see below)
- Java 11 on your local machine
- OpenShift oc cli installed on your local machine
- Appsody cli
- Docker

## Set up IBM Security Verify Tenant

IBM Cloud Identity
IBM Cloud Identity is an Identity-as-a-Service (IDaaS) and Authentication-as-a-Service (AaaS) platform. In this demonstration system it provides all of the authentication and identity services required by the demonstration application. Integration is via REST API.
If you do not have an IBM Cloud Identity tenant, you must complete at least **Exercises 1 and 2** from the [IBM Cloud Identity Basics Cookbook](https://ibm.ent.box.com/s/zyqa0qpdjfsy45q4guih09o6ehrsnif1) as a pre-requisite for this cookbook.

Once you are logged into the IBM Security Verify Tenant, you will need to add an application to your tenant. 
1. Go to Applications menu on left hand and click Add application
2. Select Custom Application and click Add application
3. Complete name, description, and company name (values don't matter)
4. Under “sign on” tab, select “sign on method” as OpenID Connect, then select the grant type as “Resource Owner Password Credentials (ROPC)” and access token format as “JWT”
5. In entitlements tab:
- Select: All users are entitled to this application
- Click Save
6. Return to Sign-on tab
-  make a note of (Application)Client ID and Client Secret
8.  Next select "API access" tab API Access tab and click Add API client
9. Complete name (value doesn't matter)
10. Select “all” for API access
- Select access token format as "JWT." 
12. Click Save
13. Select new client and edid
- Make a note of Client ID and Client Secret

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

    `./setup_databases.sh storefront-isv`

    This will create an OpenShift project call `storefront-isv` and install the databases as well as populate them with the right data.


4. Create Secret ibm-java-en

    Create secret `ibm-java-env` in the OpenShift project containing the microservices.
    ```
    oc create secret generic ibm-java-env --from-literal=KEYCLOAK_CLIENT_ID=<client-id-from-isv> --from-literal=KEYCLOAK_CLIENT_SECRET=<client-secret-from-isv> -n storefront-isv
    ```

    This secret is used by the `orders` and `catalog` microservices.

5. Deploy Inventory microservice (use `master` branch)

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
6. Deploy Catalog microservice (use `master` branch)

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

7. Deploy Orders microservice (use `isv` branch)

    In the `orders-ms-quarkus` directory run the following command:

    ```
    git checkout isv
    ./mvnw clean package -Dquarkus.kubernetes.deploy=true
    ```

    - Now create the route as follows.

    ```
    oc expose svc orders-ms-quarkus
    ```

    - To test the service see `Orders` postman collection.

8. Deploy Customer microservice (use `isv` branch)   

    In the `customer-ms-quarkus` directory run the following command:

    ```
    git checkout isv
    ./mvnw clean package -Dquarkus.kubernetes.deploy=true
    ```

    - Now create the route as follows.

    ```
    oc expose svc customer-ms-quarkus
    ```

    - To test the service, see the `Customer` postman collection.

9. Storefront UI running on OpenShift interacting with microservices on OpenShift
    
    Go to directory `storefront-ui`.

    Go to branch `hp-quarkus-version-isv`:
    ```
    git checkout hp-quarkus-version-isv
    ```

    Edit the file `config/production.json` and update the `client_secret` and the multipe `service_name` fields. The service_names will need to reference the routes for the microserices that you deployed earlier.

    Push the changes to your forked copy of the repo:

    Run the command to deploy to OpenShift:
    ```
    oc new-app --name storefront-ui https://github.com/<your-git-org>/storefront-ui#hp-quarkus-version-isv
    ```

    Expose the route
    ```
    oc expose service storefront-ui
    ```

    In a browser, go to the route.

10. Storefront UI running locally interacting with microservices on OpenShift (Optional)

    Go to directory `storefront-ui`.

    Go to branch `hp-quarkus-version-isv`:
    ```
    git checkout hp-quarkus-version-isv
    ```

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

    You can login as user `foo` pw `passw0rd`.
