# k8s template

## Required
### System
- Kubernetes
### Software to manage cert for ingress
- Cert-manager
### Cert file for pod
- *.crt
- *.key
- *.pfx

## How to create the Cert file with Bash (WSL)
 - command to generate the CRT and Key file
 ```
 $> openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out "<app_cert_name>.crt" -keyout "<app_cert_name>.key"
 ```
Sample
```
$> openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out "demo.crt" -keyout "demo.key"
```
and enter the detail like the 
 Country, State, Organization, Common name and Email.

- command to generate the PFX file from The CRT file
```
$> openssl pkcs12 -export -in demo.crt -inkey <app_cert_name>.key -out <app_cert_name>.pfx
```
Sample
```
$> openssl pkcs12 -export -in demo.crt -inkey demo.key -out demo.pfx
```
and enter the cert password if required

- command to create the secret cert to kubernetes
```
$> kubectl create secret generic <app_secret_tls> --from-file=<app_cert_name>.pfx --namespace <namespace>
```
Sample
```
$> kubectl create secret generic demo_api_tls --from-file=demo.pfx --namespace demo-ns
```
## How to install Ingress in kubernetes (power shell / bash)
if the kubernetes already ingstall ingress please go to next step
- command to add ingress to helm repo
```
$> helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
- Update helm packages
```
$> helm repo update
```
- create deployment.yaml to config installation setting
```
controller:
  config:
    compute-full-forwarded-for: "true"
    use-forwarded-headers: "true"
    proxy-body-size: "0"
  ingressClassResource:
    name: external-nginx
    enabled: true
    default: false
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - ingress-nginx
        topologyKey: "kubernetes.io/hostname"
  replicaCount: 1
  admissionWebhooks:
    enabled: false
  service:
    annotations:
      cloud.google.com/load-balancer-type: External
  metrics:
    enabled: false
  resources:
    requests:
      cpu: 500m
```
- command to install 
```
$> helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --values deployment.yaml --create-namespace
```
- To check the ingress status
Check POD
```
$> kubectl get pod -n ingress-nginx
```
Check Service
```
$> kubectl get svc -n ingress-nginx
```

# How to install cert-manager(power shell / bash)
- command to add cert-manager to helm repo
```
$> helm repo add jetstack https://charts.jetstack.io
```
- update helm repo package
```
$> helm repo update
```
- install cert-manager
```
$> helm install cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true
```
- create culster issuer from the path 'clusters/cluster-issuer.yaml' with this command
```
$> kubectl apply -f .\clusters\cluster-issuer.yaml
```

# Next . . .
Now you can implement the project with the template to work on Https port