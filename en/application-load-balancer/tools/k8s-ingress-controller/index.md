# Ingress controller for {{ managed-k8s-name }}

{{ alb-name }} provides the {{ alb-name }} Ingress controller as a tool to create and manage load balancers in [{{ managed-k8s-full-name }} clusters](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster).

After installing the controller, you will be able to use it to create an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) resource to handle incoming traffic. Based on the `Ingress` configuration, an [L7 load balancer](../../concepts/application-load-balancer.md) will be deployed automatically with all the requisite {{ alb-name }} resources.

## Sample configuration {#example}

Below is an abbreviated sample configuration for an `Ingress` resource. It will be used to create a load balancer to receive HTTPS traffic and to distribute it to two services based on the URI request path.

{% cut "Example" %}

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-demo-tls
spec:
  tls:
    - hosts:
        - <domain_name>
      secretName: yc-certmgr-cert-id-<certificate_ID>
  rules:
    - host: <domain_name>
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: alb-demo-1
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: alb-demo-2
                port:
                  number: 80
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: alb-demo-2
                port:
                  name: http
```

Where `secretName` is the ID of the certificate from {{ certificate-manager-full-name }}.

{% endcut %}

## Logging settings {#log-options}

To set up logging for an L7 load balancer created using an {{ alb-name }} Ingress controller in your {{ managed-k8s-name }} cluster, add the `ingress.alb.yc.io/group-settings-name` annotation with a name for the Ingress group settings and specify the settings in the additional `IngressGroupSettings` resource. For more information, see [Ingress resource annotations](../../k8s-ref/ingress.md#annotations).

## Installation and requirements {#install}

To install the {{ alb-name }} Ingress controller, you need:

* {{ managed-k8s-name }} cluster.
* Cluster node group.
* Cluster namespace to store the [service account](service-account.md) key.

To learn how to install the Ingress controller, see [this guide](../../operations/k8s-ingress-controller-install.md).

#### See also {#see-also}

* [Controller operating principles](principles.md).
* [Security group configuration](security-groups.md) for a {{ k8s }} cluster and a load balancer.
* [Service account](service-account.md) to support controller operation.