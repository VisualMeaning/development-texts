# Manually generating SSL certs for Azure web apps using Certbot

## Local operations on the CLI

### Generate the cert

Install `certbot`:

    sudo apt install certbot python3-certbot-nginx

Generate a cert for a target domain using DNS challenge type:

    sudo certbot certonly --manual --preferred-challenges dns -d domain.com

This will tell you to add a DNS TXT record for that domain. Most of our domains are now managed through Amazon Route53, so
go there in your browser, find the Hosted Zone that corresponds to the domain you're generating the cert for, and then add
the TXT record.

Once the TXT record is saved, tell certbot to continue. It should report success and save your certificate and signing chain to
`/etc/letsencrypt/live/domain.com`

### Alternatively...

There's a plugin that lets us automate doing the TXT record faffing in Route53. Install with:

    sudo apt install python3-certbot-dns-route53

This uses standard AWS CLI credentials, but since certbot needs to be run as root, we need to symlink our local user's AWS
credentials directory to an equivalent location for root:

    sudo -s
    ln -s /home/alightwing/.aws ~/.aws
    exit

Then if we want to renew the cert, all we need to do is:

    sudo certbot certonly --dns-route53 -d domain.com

This won't renew the cert if it's not close to expiring. To force a renewal:

    sudo certbot certonly --dns-route53 -d domain.com --force-renewal

### Convert Let's Encrypt cert files to Private Certificate

Azure cert attachments need to come in the form of Private Key Certificates (.pfx) so we need to convert our .pem files from 
Let's Encrypt to .pfx. This is done with `openssl`:

    sudo openssl pkcs12 -export -out domain.com_fullchain.pfx -inkey /etc/letsencrypt/live/domain.com/privkey.pem -in /etc/letsencrypt/live/domain.com/fullchain.pem

On Ubuntu/other Linux OS, you will also have to `chown` the file so that your browser can access it, as otherwise it will be 
owned by root.

    chown <username>:<username> domain.com_fullchain.pfx

where `username` is your Linux username.

## In the Azure Portal

### Add custom domain (one time only)

If you haven't yet added a custom domain in Azure for the domain you're trying to add the SSL cert for, go to the Web App in the
 Azure Portal and open "Custom Domains" in the left panel.

Click "Add custom domain" and enter the domain you want to add.

This will give you instructions for another DNS challenge. Add the TXT and CNAME records specified, and then click Validate.

This should add the domain to the list of custom domains for this Web App

### Upload and bind SSL certificate

Go to the Web App in the Azure Portal and open the "TLS/SSL settings" menu from the left panel.

Click on "Private Key Certificates (.pfx)" and upload the .pfx file.

Then click on "Bindings" and then "Add TLS/SSL Binding". Select the correct custom domain and the corresponding Private 
Certificate. Select `SNI SSL` as the TLS/SSL type, and then click "Add Binding".
