# Manually generating SSL certs for Azure web apps using Certbot

## Local operations on the CLI

### Generate the cert

Install `certbot` and the route53 plugin:

    sudo apt install certbot python3-certbot-dns-route53

The plugin can use the standard boto ways of obtaining credentials, but certbot wants to run as root, so need a step for the root user to know where to find your credentials file. Easiest with an environment variable and the `-E` flag for sudo.

    export AWS_SHARED_CREDENTIALS_FILE=~/.aws/credentials

Then if we want to renew the cert, all we need to do is:

    sudo -E certbot certonly --dns-route53 -d $DOMAIN

This won't renew the cert if it's not close to expiring. To force a renewal:

    sudo -E certbot certonly --dns-route53 -d $DOMAIN --force-renewal

#### Or instead, manual fallback

Generate a cert for a target domain using DNS challenge type:

    sudo certbot certonly --manual --preferred-challenges dns -d $DOMAIN

This will tell you to add a DNS TXT record for that domain. Most of our domains are now managed through Amazon Route53, so go there in your browser, find the Hosted Zone that corresponds to the domain you're generating the cert for, and then add the TXT record.

Once the TXT record is saved, tell certbot to continue. It should report success and save your certificate and signing chain to `/etc/letsencrypt/live/$DOMAIN`

### Convert Let's Encrypt cert files to Private Certificate

Azure cert attachments need to come in the form of Private Key Certificates (.pfx) so we need to convert our .pem files from Let's Encrypt to .pfx. This is done with `openssl`:

    sudo openssl pkcs12 -export -out ${DOMAIN}_fullchain.pfx -inkey /etc/letsencrypt/live/${DOMAIN}/privkey.pem -in /etc/letsencrypt/live/${DOMAIN}/fullchain.pem -legacy

Note `-legacy` is required with openssl 3 as Azure's requirements for certificates don't match the new encryption defaults.

The command will prompt (twice) to enter an "Export Password" to make the output key non-plaintext. The password could be blank, but the upload form on Azure will not submit with an empty value, so supply one.

On Ubuntu/other Linux OS, you will also have to `chown` the file so that your browser can access it, as otherwise it will be owned by root.

    sudo chown $USER:$USER ${DOMAIN}_fullchain.pfx

## In the Azure Portal

### Add custom domain (one time only)

If you haven't yet added a custom domain in Azure for the domain you're trying to add the SSL cert for, go to the Web App in the Azure Portal and open "Custom domains" in the left panel.

Click "Add custom domain" and enter the domain you want to add.

This will give you instructions for another DNS challenge. Add the TXT and CNAME records specified, and then click Validate.

This should add the domain to the list of custom domains for this Web App

### Upload and bind SSL certificate

Go to the Web App in the Azure Portal and open the "Certificates" menu from the left panel.

Click on "Bring your own certificates (.pfx)" and in the dialog for "Source" select "Upload certificate (.pfx)" and select the file, add password, and name. Then "Validate" the name and "Add" to upload the cert. Provided the cert is in the right format, it will then be created.

Next under "Custom domains" in the left panel, select the domain and then "update binding" under the triple dot button in the row. The "Certificate" field in the panel should let you select cert by the name chosen, and "TLS/SSL type" should be "SNI SSL", then "Update" to complete.

Finally verify the domain by loading a page and using browser tools to look at the cert, and notify from expiry when update is next required.
