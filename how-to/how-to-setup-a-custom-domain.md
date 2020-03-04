# How to set up a custom domain

## Introduction

Skygear provides default suffixed domain `.skygearapp.com` for you to access your application. You can update your application to use a custom domain which owned by you instead of the skygear generated domain. 

More commands on custom domains can be found [here](../skycli/list-of-commands.md#skycli-domain).

## Prerequisites

To setup custom domain, you need to install and configure `skycli` properly. Please follow [this guide](../set-up/set-up-steps.md) to setup skycli.

## Setup custom domain

### 1. Add custom domain

```bash
$ skycli domain add <your_domain>

#example
$ skycli domain add www.example.com --app=myapp
```

### 2. Verify domain ownership

#### Add DNS records in your DNS provider

After adding the custom domain, please follow the instructions shown in the console and add dns records in your dns provider. 

```bash
#exmaple
$ skycli domain add www.example.com --app=myapp
Added domain www.example.com successfully!

Add following DNS records in your DNS provider.

TYPE      HOST                     VALUE
TXT       _skygear.example.com     <txt_record_value>
A         www.example.com          <a_record_value>

After updating DNS records, run `skycli domain verify www.example.com` to verify domain.
```

####  Ask Skygear Cluster to verify the domain

```bash
$ skycli domain verify <your_domain>

#example
$ skycli domain verify www.example.com --app=myapp
Success! You can now access your app through www.example.com.
Your site may show a security certificate warning until the certificate has been provisioned.
```

After verifying the domain successfully, you can now access your application with the new domain.

### 3. \(Optional\) Setup custom certificate

Skygear Cluster by default provisions SSL certificate signed by [Let's Encrypt](https://letsencrypt.org/). If you want to use your own certificate, you can update it with `skycli domain update` command.

```bash
# Create tls secret
$ skycli secret create <your_tls_secret_name> --type=tls --cert=<path/to/tls-cert.pem> --key=<path/to/tls-key.pem>
# Update domain to use tls secret
$ skycli domain update <your_domain> --tls-secret=<your_tls_secret_name>


#example
$ skycli secret create myapp-tls --type=tls --cert=path/to/tls-cert.pem --key=path/to/tls-key.pem --app=myapp
Success! Created secret myapp-tls

$ skycli domain update example.com --tls-secret=myapp-tls --app=myapp
Success! Updated domain example.com

$ skycli domain list --app=myapp
DOMAIN            VERIFIED         CUSTOM_CERT      REDIRECT            SSL_CERT_EXPIRY               CREATED_AT
example.com       true             true             -                   2020-11-26 20:00:00 +08:00    2020-01-01 18:00:00 +08:00
```

### 4. \(Optional\) Setup redirect

For some case, you may want to redirect all requests of a custom domain to another specific domain \(e.g. Redirect `example.com`  to `www.example.com`\).

```bash
$ skycli domain update example.com --redirect-domain=www.example.com
Success! Updated domain example.com

$ skycli domain list --app=myapp
DOMAIN              VERIFIED     CUSTOM_CERT      REDIRECT            SSL_CERT_EXPIRY               CREATED_AT
www.example.com     true         false            -                   -                             2019-11-26 18:00:00 +08:00
example.com         true         false            www.example.com     -                             2019-11-26 18:00:00 +08:00
```



