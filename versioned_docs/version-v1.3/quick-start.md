---
title: Deploy First Application
---

> Before starting, please confirm that you've installed KubeVela and enabled the VelaUX addon according to [the installation guide](./install.mdx).

Welcome to KubeVela! This section will guide you to deliver your first app.

## Deploy a classic application via CLI

Below is a classic KubeVela application which contains one component with one operational trait, basically, it means to deploy a container image as webservice with one replica. Additionally, there are three policies and workflow steps, it means to deploy the application into two different environments with a bit different configurations.

```yaml
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: first-vela-app
spec:
  components:
    - name: express-server
      type: webservice
      properties:
        image: crccheck/hello-world
        ports:
         - port: 8000
           expose: true
      traits:
        - type: scaler
          properties:
            replicas: 1
  policies:
    - name: target-default
      type: topology
      properties:
        # The cluster with name local is installed the KubeVela.
        clusters: ["local"]
        namespace: "default"
    - name: target-prod
      type: topology
      properties:
        clusters: ["local"]
        # This namespace must be created before deploying.
        namespace: "prod"
    - name: deploy-ha
      type: override
      properties:
        components:
          - type: webservice
            traits:
              - type: scaler
                properties:
                  replicas: 2
  workflow:
    steps:
      - name: deploy2default
        type: deploy
        properties:
          policies: ["target-default"]
      - name: manual-approval
        type: suspend
      - name: deploy2prod
        type: deploy
        properties:
          policies: ["target-prod", "deploy-ha"]
```

* Starting deploy the application

```bash
# This command for creating a namespace in the local cluster
$ vela env init prod --namespace prod
$ vela up -f https://kubevela.net/example/applications/first-app.yaml
```

* View the process and status of the application deploy

```bash
$ vela status first-vela-app
```

The application will become a `workflowSuspend` status if the first step is successfully run.

* Resume the workflow

```bash
$ vela workflow resume first-vela-app
```

* Access the application

```bash
$ vela port-forward first-vela-app 8000:8000
<xmp>
Hello World


                                       ##         .
                                 ## ## ##        ==
                              ## ## ## ## ##    ===
                           /""""""""""""""""\___/ ===
                      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
                           \______ o          _,/
                            \      \       _,'
                             `'--.._\..--''
</xmp>
```

Great! You have finished deploying your first KubeVela application, the simplest component can only have one component, the rest fields are all optional including trait, policies and workflow.

> Currently, The application created by CLI will be synced to UI, but it will be readonly.

## Deploy a simple application via UI

After logging into the UI, the first page you enter is for managing the applications:

Then click the button of `New Application` on the upper-right, type in these things:

1. Name and other basic Infos.
2. Choose the Project. We've created a default Project for you to use or you can click `New` to create your own.
3. Choose the main component type. In this case, we use `webservice` to deploy Stateless Application.
4. Choose your environment. We select the `Default` Environment based on the `Default` Target.

### Setting up properties

Next step, we see the page of properties. Configure following:

- Image address `crccheck/hello-world`

![create hello world app](https://static.kubevela.net/images/1.3/create-helloworld.jpg)

Confirmed. Notice that this application is only created but not deployed yet. VelaUX default generates [Workflow](./getting-started/core-concept#workflow) and a scaler [Trait](./getting-started/core-concept#trait).

### Executing Workflow to deploy

Click the deploy button on the upper-right. When the workflow is finished, you'll get to see the list of status lying within.

![](./resources/succeed-first-vela-app.jpg)

## Deleting Application

If you want to delete the application when it's no longer used, simply:

1. Enter the page of environment, click `Recycle` to reclaim the resources that this environment used.
2. Go back to the list of applications and click the drop-down menu to remove it.

That's it! You succeed at the first application delivery. Congratulation!

## Next Step

- View [Core Concepts](./getting-started/core-concept) to look on more concepts.
- View [User Guide](./tutorials/webservice) to look on more of what you can achieve with KubeVela.
