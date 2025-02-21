---
title: Expose an external application
---

This example shows how we can expose a service located outside the Kubernetes cluster using an Ingress.

## Installation

Please follow the [deployment](/kubernetes-ingress-controller/{{page.kong_version}}/deployment/overview) documentation to install
the {{site.kic_product_name}} on your Kubernetes cluster.

## Testing Connectivity to Kong

This guide assumes that the `PROXY_IP` environment variable is
set to contain the IP address or URL pointing to Kong.
Please follow one of the
[deployment guides](/kubernetes-ingress-controller/{{page.kong_version}}/deployment/overview) to configure this environment variable.

If everything is setup correctly, making a request to Kong should return
HTTP 404 Not Found.

```bash
$ curl -i $PROXY_IP
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.2.1

{"message":"no Route matched with those values"}
```

This is expected as Kong does not yet know how to proxy the request.

## Create a Kubernetes service

First we need to create a Kubernetes Service [type=ExternalName][0] using the hostname of the application we want to expose.

```bash
echo "
kind: Service
apiVersion: v1
metadata:
  name: proxy-to-httpbin
spec:
  ports:
  - protocol: TCP
    port: 80
  type: ExternalName
  externalName: httpbin.org
" | kubectl create -f -
```

## Create an Ingress to expose the service at the path `/foo`

```bash
echo '
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: proxy-from-k8s-to-httpbin
  annotations:
    konghq.com/strip-path: "true"
spec:
  ingressClassName: kong
  rules:
  - http:
      paths:
      - path: /foo
        pathType: ImplementationSpecific
        backend:
          service:
            name: proxy-to-httpbin
            port:
              number: 80
' | kubectl create -f -
```

## Test the service

```bash
$ curl -i $PROXY_IP/foo
```

[0]: https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors
