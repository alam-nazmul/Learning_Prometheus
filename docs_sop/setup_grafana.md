## How to Install Grafana on AlmaLinux 9


Before we start with Grafana installation, we need to update the system packages to the latest version available.
```angular2html
dnf update -y
dnf upgrade -y
```

Before we start with the installation, we need to add first, the Grafana repository since it is not added in AlmaLinux 9 by default. To do that, create the following file:
```angular2html
vim /etc/yum.repos.d/grafana.repo
```

Paste the following lines of code:
```angular2html
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```


Save the file, close it, and update the system.
```angular2html
dnf update -y
```

Once the system is updated and the port is opened, install Grafana with the following command:
```angular2html
dnf install grafana -y
```


Start and enable the service after successful installation:

```angular2html
systemctl enable grafana-server --now
```


To install the Apache web server execute the following command:
```angular2html
dnf install httpd -y
```


Once installed, start and enable the service.
```angular2html
systemctl enable httpd --now
```

We need to create an Apache virtual host configuration file in order can access Grafana via the domain name:
```angular2html
vim /etc/httpd/conf.d/grafana.conf
```

Paste the following lines of code:
```angular2html
<VirtualHost *:80>
     ServerName yourdomain.com
     DocumentRoot /var/www/html/
     <Directory /var/www/html/>
          Options FollowSymlinks
          AllowOverride All
          Require all granted
     </Directory>

    ProxyRequests off 
    ProxyPass / http://127.0.0.1:3000/ 
    ProxyPassReverse / http://127.0.0.1:3000/

     ErrorLog /var/log/httpd/yourdomain.com_error.log
     CustomLog /var/log/httpd/yourdomain.com.log combined
</VirtualHost>
```

Save the file, close it, and check the syntax of the Apache configuration file.
```angular2html
httpd -t
```




Allow port 3000 in the firewall. This is the port that is Grafana service is running on.
```angular2html
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --reload
```


Add Zabbix server with Grafana, need to install required plugins
```angular2html
grafana-cli plugins install alexanderzobnin-zabbix-app
grafana-cli plugins install grafana-piechart-panel
```


Restart the httpd.service.
```angular2html
systemctl restart httpd
```



Finally, you can access the Grafana via the domain name at http://Yourdomain.com



Enter the credentials as below
```angular2html
user: admin
password: admin
```

Go to Administrator > Plugins and data > Plugins

Search "Zabbix" and Enable it.


Then Go to Connections > Data sources > Zabbix

Put the credentials


### How to integrate Prometheus with Grafana

Then Go to Connections > Data sources > Prometheus


### How to configure the Grafana with a domain

So, we have to modify the Grafana init file as below

```
vim /etc/grafana/grafana.ini
```

Paste the following lines of code:

```
    [server]
    # Protocol (http, https, h2, socket)
    http_addr = 10.10.10.15
    http_port = 3000
    domain = mydomain
    root_url = https://mydomain/obs/
    serve_from_sub_path = false
    enforce_domain = true

    [auth]
    login_maximum_inactive_lifetime_duration = 15m
    login_maximum_lifetime_duration = 6h
    logout_redirect_url = /login
    logout_redirect_url = https://mydomain/obs/login

    [security]
    cookie_secure = true
    cookie_samesite = lax  
```


Restart the Grafana server service

```
systemctl restart grafana-server
```