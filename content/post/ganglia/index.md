
```sh
yum install httpd ganglia-gmetad ganglia-gmond ganglia-web
```
[防火墙放行httpd](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7)：
```sh
sudo firewall-cmd --permanent --add-service=http
```

