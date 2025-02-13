---
id: service
---

# Expose Service

You can determine how to expose your service in the `AppConfiguration` model via the `ports` field (under the `network` accessory). The `ports` field defines a list of all the `Port`s you want to expose for the application (and their corresponding listening ports on the container, if they don't match the service ports), so that it can be consumed by other applications.

Unless explicitly defined, each of the ports exposed is by default exposed privately as a `ClusterIP` type service. You can expose a port publicly by specifying the `public` field in the `Port` schema. At the moment, the implementation for publicly access is done via Load Balancer type service backed by cloud providers. Ingress will be supported in a future version of kusion.

For the `Port` schema reference, please see [here](../../../6-reference/2-modules/1-developer-schemas/workload/service.md#schema-port) for more details.

## Prerequisites

Please refer to the [prerequisites](1-deploy-application.md#prerequisites) in the guide for deploying an application.

The example below also requires you to have [initialized the project](1-deploy-application.md#initializing) using the `kusion workspace create` and `kusion init` command, which will create a workspace and also generate a [`kcl.mod` file](1-deploy-application.md#kclmod) under the stack directory.

## Managing Workspace Configuration

In the first guide in this series, we introduced a step to [initialize a workspace](1-deploy-application.md#initializing-workspace-configuration) with an empty configuration. The same empty configuration will still work in this guide, no changes are required there.

However, if you (or the platform team) would like to set default values for the services to standardize the behavior of applications in the `dev` workspace, you can do so by updating the `~/dev.yaml`:

```yaml
modules:
  kusionstack/network@0.1.0: 
    default:
      port: 
        type: alicloud
        labels:
            kusionstack.io/control: "true"
        annotations:
            service.beta.kubernetes.io/alibaba-cloud-loadbalancer-spec: slb.s1.small
```

The workspace configuration need to be updated with the command:

```bash
kusion workspace update dev -f ~/dev.yaml
```

For a full reference of what can be configured in the workspace level, please see the [workspace reference](../../../6-reference/2-modules/2-workspace-configs/networking/network.md).

## Example

`simple-service/dev/main.k`:
```python
import kam.v1.app_configuration as ac
import service
import service.container as c
import network as n

"helloworld": ac.AppConfiguration {
    workload: service.Service {
        containers: {
            "helloworld": c.Container {
                image = "gcr.io/google-samples/gb-frontend:v4"
                env: {
                    "env1": "VALUE"
                    "env2": "VALUE2"
                }
                resources: {
                    "cpu": "500m"
                    "memory": "512Mi"
                }
                # Configure an HTTP readiness probe
                readinessProbe: p.Probe {
                    probeHandler: p.Http {
                        url: "http://localhost:80"
                    }
                    initialDelaySeconds: 10
                }
            }
        }
        replicas: 2
    }
    accessories: {
        "network": n.Network {
            ports: [
              n.Port {
                  port: 8080
                  targetPort: 80
              }
            ]
        }
    }
}
```

The code above changes the service port to expose from `80` in the last guide to `8080`, but still targeting the container port `80` because that's what the application is listening on.

## Applying

Re-run steps in [Applying](1-deploy-application.md#applying), new service configuration can be applied.

```
$ kusion apply
 ✔︎  Generating Spec in the Stack dev...                                                                                                                                                                                                     
Stack: dev  ID                                                               Action
* ├─     v1:Namespace:simple-service                                      UnChanged
* ├─     v1:Service:simple-service:simple-service-dev-helloworld-private  Update
* └─     apps/v1:Deployment:simple-service:simple-service-dev-helloworld  UnChanged


? Do you want to apply these diffs? yes
Start applying diffs ...
 SUCCESS  UnChanged v1:Namespace:simple-service, skip                                                                                                                                                                                         
 SUCCESS  Update v1:Service:simple-service:simple-service-dev-helloworld-private success                                                                                                                                                      
 SUCCESS  UnChanged apps/v1:Deployment:simple-service:simple-service-dev-helloworld, skip                                                                                                                                                     
UnChanged apps/v1:Deployment:simple-service:simple-service-dev-helloworld, skip [3/3] ██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████ 100% | 0s
Apply complete! Resources: 0 created, 1 updated, 0 deleted.
```

## Validation

We can verify the Kubernetes service now has the updated attributes (mapping service port 8080 to container port 80) as defined in the `ports` configuration:

```
kubectl get svc -n simple-service -o yaml
...
  spec:
    ...
    ports:
    - name: simple-service-dev-helloworld-private-8080-tcp
      port: 8080
      protocol: TCP
      targetPort: 80
...
```

Exposing service port 8080:
```
kubectl port-forward svc/simple-service-dev-helloworld-private -n simple-service 30000:8080
```

Open browser and visit [http://127.0.0.1:30000](http://127.0.0.1:30000), the application should be up and running：

![app-preview](/img/docs/user_docs/guides/working-with-k8s/app-preview.png)