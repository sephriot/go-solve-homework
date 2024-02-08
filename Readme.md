# An exemplary helm chart for EKS based service

## Requirements

- Accessible to the internet on ports 80 and 443 with certificate for 443
- Accessible internally on ports 80 and 443 with certificate for 443 (the
  application will have more features for local IP ranges such as admin panels)
- Has access to secrets as environment variables: DB_HOST, DB_UN, DB_PW
- Has access to a volume to store large amounts of temporary data
- Has traffic shaping for the paths /store and /blog which go to other
  services. This should happen prior to hitting the service.
- Requires that a template is downloaded and hydrated with secrets (envsubst or
  similar) and then made available to the app at /etc/opt/fancyapp/config.

## Assumptions

Helm chart assumes there is an nginx-ingress controller deployed and is running using dual load balancer mode (external + internal).

```
controller:
  ingressClassByName: true

  ingressClassResource:
    name: nginx-internal
    enabled: true
    default: false
    controllerValue: "k8s.io/ingress-nginx-internal"

  service:
    # Disable the external LB
    external:
      enabled: false

    # Enable the internal LB. The annotations are important here, without
    # these you will get a "classic" loadbalancer
    internal:
      enabled: true
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
        service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: 'true'
        service.beta.kubernetes.io/aws-load-balancer-type: nlb
```

[Reference](https://stackoverflow.com/questions/67203957/how-to-create-only-internal-load-balancer-with-ingress-nginx-chart)

Another assumption is that there is a cert manager deployed that provides and manages certificates on demand.
