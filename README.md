# k8s-traefik-certmanager

A simple how-to configure Traefik on Kubernetes with Certmanager using Letsencrypt for issuing certificates. Tested on GKE.

I'm using Letsencrypt's HTTP01 verification so your hosts must resolve. Other validation methods and providers work just fine.

This example assumes you intend to use an external loadBalancerIP ingress. (In GCP this is a _Regional_ static address.)

## Why Certmanager

Certmanager stores what it needs in etcd instead of Traefiks default method of using persistent volumes. This lets you run multiple replicas on separate nodes if your k8s platform doesn't support `ReadWriteMany` persistent volumes.

## Contributing

This was just something I threw together, contributions for improving it / testing on other platforms, etc. are gladly welcome. 

I don't plan on making this into a helm chart.

## 1. Install Cert Manager

(Following [Installing with regular manifests](https://cert-manager.io/docs/installation/kubernetes/#installing-with-regular-manifests)):

```bash
$ kubectl create namespace cert-manager
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.13.1/cert-manager.yaml
$ kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=$(gcloud config get-value core/account)
```

Verify it's running:

```bash
$ kubectl get pods --namespace cert-manager
```

## 2. Edit and apply the Certmanager manifests

1. Modify `cert-manager/00-clusterIssuer.yaml` replace `{{replace_me@example.com}}` with your email address.
1. `kubectl apply -f cert-manager/00-clusterIssuer.yaml`

## 3. Edit and apply the Traefik manifests

1. Modify `traefik/02-service.yaml` replace `{{replace_me}}` with your external IP address.
1. (Optional) Modify `traefik/04-deployment.yaml`. Change whatever you'd like. The resource limits are tuned to my environment. YMMV.
1. `kubectl apply -f traefik`
1. Verify traefik is now running: `kubectl get pods --namespace traefik

## 4. (Optional) Test with httpbin (httpbin.org clone)

1. Modify `httpbin/01-certificate.yaml`
   1. Replace every `{{replace this line}}` with your hostname. **IMPORTANT**: This must be mapped to receive a TLS Certificate!
1. Modify `httpbin/04-traefik-ingress.yaml`
    1. Modify the `match` rules with your hostname.
    1. Replace the `secretName` to match the secret name defined in `httpbin/01-certificate.yaml`
1. `kubectl apply -f httpbin`
1. Verify the pod is running: `kubectl get pods --namespace httpbin`
1. Wait about 10-15 minutes.
1. Navigate to httpbin.example.com and https://httpbin.example.com 

## 5. Success

Hopefully by this step you will have httpbin running with a valid TLS certificate.

## Known Issues

* I have not been able to figure out how to get the dashboard working, it hasn't been a big deal as I don't rely on it.
* I haven't been able to figure out how to correctly forward the remote IP Address
