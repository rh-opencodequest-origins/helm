Helm charts for Platform Engineering Workshop.


To verify the RHTAS installation & configuration:

1) Login to the OpenShift Cluster

2) Download `cosign` from the OCP console / "Command Line Tools" and install it in your PATH

3) `oc project trusted-artifact-signer` 

4) Setup your environment

```
export TUF_URL=$(oc get tuf -o jsonpath='{.items[0].status.url}' -n trusted-artifact-signer)
export OIDC_ISSUER_URL=https://$(oc get route keycloak -n keycloak | tail -n 1 | awk '{print $2}')/realms/backstage
export COSIGN_FULCIO_URL=$(oc get fulcio -o jsonpath='{.items[0].status.url}' -n trusted-artifact-signer)
export COSIGN_REKOR_URL=$(oc get rekor -o jsonpath='{.items[0].status.url}' -n trusted-artifact-signer)
export COSIGN_MIRROR=$TUF_URL
export COSIGN_ROOT=$TUF_URL/root.json
export COSIGN_OIDC_CLIENT_ID="trusted-artifact-signer"
export COSIGN_OIDC_ISSUER=$OIDC_ISSUER_URL
export COSIGN_CERTIFICATE_OIDC_ISSUER=$OIDC_ISSUER_URL
export COSIGN_YES="true"
export SIGSTORE_FULCIO_URL=$COSIGN_FULCIO_URL
export SIGSTORE_OIDC_ISSUER=$COSIGN_OIDC_ISSUER
export SIGSTORE_REKOR_URL=$COSIGN_REKOR_URL
export REKOR_REKOR_SERVER=$COSIGN_REKOR_URL
```

5) `cosign initialize` 

6) Sign an arbitrary container image, for example

```
echo "FROM scratch" > ./tmp.Dockerfile
podman build . -f ./tmp.Dockerfile -t ttl.sh/rhtas/test-image:1h

podman push ttl.sh/rhtas/test-image:1h
```

`cosign sign -y ttl.sh/rhtas/test-image:1h` 

When asked to authenticate, use one of the registered workshop users/password, e.g. `dev1@rhdemo.com` 


7) Verify the signature

`cosign verify --certificate-identity=dev1@rhdemo.com ttl.sh/rhtas/test-image:1h`

(add `| jq` to make it readable)

8) Show signature/security info related to the OCI artifact

`cosign tree ttl.sh/rhtas/test-image:1h`

