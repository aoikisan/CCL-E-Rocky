# CCL-E-Rocky
Cockpit Certbot Let's Encrypt Rocky Linux


Cockpit works on a **web socket combined with http/https** interface, Web Socket is used to deliver active content back and forth between client and server. But when a proxy sits in between, it needs to be configured likely.

## Set up Apache Virtual Host

Install Apache web server with the following command:
<br>
`sudo dnf -y install httpd`

Run the following command to create an Apache virtual host file. Replace the domain name with your actual domain name for Cockpit. Don’t forget to create an A record for this domain name.
<br>
`sudo vi /etc/httpd/conf.d/cockpit.domainName.conf`

Put the following text into the file.
<br>
```
<VirtualHost *:80>
 ServerName cockpit.domainname.com
</VirtualHost>
```

Then restart Apache.
<br>
`sudo systemctl restart httpd`


## Securing your Cockpit
## Will be using Let's Encrypt with Certbot

#Certbot
#Snapd component
<br>
`sudo dnf install epel-release`
`sudo dnf upgrade`

`sudo yum install snapd`
`sudo systemctl enable --now snapd.socket`
<br>
#classic support enabled
<br>
`sudo ln -s /var/lib/snapd/snap /snap`
#logout and or reboot, do this or you will run into funky issues, and troubleshoot for no reason your call

#Certbot Install
<br>
`dnf install -y certbot python3-certbot-apache`
`systemctl restart httpd`

#Run and install certs for domain
#Did you get a bug when you ran this command, run it again you goof
<br>
`certbot --apache -d cockpit.domainname.com`

#So good news, is now you will have a SSL CONF file
#Lets now modify the CONF file so it that some of the data
<br>
```
<IfModule mod_ssl.c>
<VirtualHost *:443>
  ServerName cockpit.domainname.com
  SSLCertificateFile /etc/letsencrypt/live/cockpit.domainname.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/cockpit.domainname/privkey.pem
  Include /etc/letsencrypt/options-ssl-apache.conf

  ProxyPreserveHost On
  ProxyRequests Off

  # allow for upgrading to websockets
  RewriteEngine On
  RewriteCond %{HTTP:Upgrade} =websocket [NC]
  RewriteRule /(.*)           ws://127.0.0.1:9090/$1 [P,L]
  RewriteCond %{HTTP:Upgrade} !=websocket [NC]
  RewriteRule /(.*)           http://127.0.0.1:9090/$1 [P,L]

SSLProtocol TLSv1.3
SSLCipherSuite TLSv1.3 TLS_AES_256_GCM_SHA384

  # Proxy to your local cockpit instance
  ProxyPass / http://127.0.0.1:9090/
  ProxyPassReverse / http://127.0.0.1:9090/

</VirtualHost>
</IfModule>
```

Save and close the file. Then restart Apache web server.
<br>
`sudo systemctl restart httpd`

Make sure you changed `/etc/cockpit/cockpit.conf` to include the following:
<br>
```
[WebService]
Origins = https://cockpit.your-domain.com http://127.0.0.1:9090
ProtocolHeader = X-Forwarded-Proto
AllowUnencrypted = true
```

Restart cockpit
<br>
`sudo systemctl restart cockpit.service`

You can now log in to cockpit using your browser.
