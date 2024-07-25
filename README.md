# Overview
A 10 minute demo to introduce Akeyless

## Generate a Customer Fragment

```bash
akeyless gen-customer-fragment --name CustomerFragmentDemo --description MyFirstCF --json > customer_fragments.json
```

You'll get the following output:

```bash
{
  "customer_fragments": [
    {
      "id": "cf-p6lta4na60yzkmwrjvtm",
      "value": "naTTMPLA558l934TZCnK6E0jItSaeVbtp+1MtGnCKsLHOrxi0YW1E6K88EUTwWCVMyt4VTDjmj7D/UssLlGCeA==",
      "description": "MyFirstCF",
      "name": "CustomerFragmentDemo"
    }
  ]
}
```



## Create a Gateway

```bash
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 5696:5696 -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json -e ADMIN_ACCESS_ID="sam.gabrail@tekanaid.com" -e ADMIN_ACCESS_KEY="" --name akeyless-codespaces-gw akeyless/base:latest-akeyless
```

## For K8s Gateway

Create a base64 encoding of the customer fragment:

```bash
base64 -w 0 customer_fragments.json
```

and add that to the `customer-fragments-secret.yaml` file.

Now apply this secret to our namespace

```bash
kubectl apply -f customer-fragments-secret.yaml
```

Update the `values.yaml` file to reference this secret:

```yaml
customerFragmentsExistingSecret: customer-fragments-secret
```

I've added a certificate and private key in the values.yaml file. I created the cert using Akeyless.

Install with helm

```bash
helm repo add akeyless https://akeylesslabs.github.io/helm-charts
helm repo update
helm upgrade --install akeyless-gw --namespace akeyless akeyless/akeyless-api-gateway -f values.yaml
```


## Generate a Certificate for the Gateway

### With Akeyless
I tried the below and failed to get it to work.

You need to create a classic key and import a private key that you can create with: 
```bash
openssl genpkey -algorithm RSA -out private.key
```

I created the key: `/Certs/GWY_CSR_KEY` by first creating the private key above and importing it. This way I can still access the private key. If I get akeyless to create the private key I can't see it after that. But I need it to add it to the `values.yaml` file along with the certificate.

Then I need to create a CSR using akeyless as shown below:

```bash
akeyless generate-csr --name /Certs/GWY_CSR_KEY --ip-addresses "192.168.1.82,192.168.1.83,192.168.1.84,127.0.0.1" --common-name tekanaid.com
```

Then finally I can use the `PKI_Issuer` to generate a certificate with the CSR created and the private key. Then use the cert and private key in the `values.yaml` file.

The problem is that the ip-addresses don't appear in the final certificate.

### With Openssl

I ended up using Openssl using the following steps:

```bash
cd baremetal-k3s-gateway-config
# Create the CSR
openssl req -new -key private.key -out request.csr -config openssl.cnf
# Create the Certificate
openssl x509 -req -in request.csr -signkey private.key -out certificate.crt -days 1000 -extfile openssl.cnf -extensions req_ext
```

I then uploaded this certificate into Akeyless to monitor it and alert me when it is close to expire.

## Zero Knowledge with Akeyless CLI

In case you are working with your own Fragment, please create an environment variable AKEYLESS_GATEWAY_URL to point your CLI to interact with the relevant Gateway e.g:

Linux OS :export AKEYLESS_GATEWAY_URL=<https://Your_GW_URL:8080>

## Gateway with a Self-Signed Certificate

In case your Gateway using a self-signed certificate which is not trusted on your machine, set the environment variable AKEYLESS_TRUSTED_TLS_CERTIFICATE_FILE with your certificate pem file location. Alternitivly you can simply install your certificate on the machine.