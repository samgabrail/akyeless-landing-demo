# Overview
A 10 minute demo to introduce Akeyless

## Generate a Customer Fragment

```bash
akeyless gen-customer-fragment --name CodeSpacesFragment --description MyFirstCF --json > customer_fragments.json
```

You'll get the following output:

```bash
{
  "customer_fragments": [
    {
      "id": "cf-p6lta4na60yzkmwrjvtm",
      "value": "naTTMPLA558l934TZCnK6E0jItSaeVbtp+1MtGnCKsLHOrxi0YW1E6K88EUTwWCVMyt4VTDjmj7D/UssLlGCeA==",
      "description": "MyFirstCF",
      "name": "CodeSpacesFragment"
    }
  ]
}
```



## Create a Gateway

```bash
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 5696:5696 -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json -e ADMIN_ACCESS_ID="sam.gabrail@tekanaid.com" -e ADMIN_ACCESS_KEY="" --name akeyless-codespaces-gw akeyless/base:latest-akeyless
```

docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 5696:5696 -e ADMIN_ACCESS_ID="sam.gabrail@tekanaid.com" -e ADMIN_ACCESS_KEY="" --name akeyless-codespaces-gw akeyless/base:latest-akeyless
