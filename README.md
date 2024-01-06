# common-maven
common maven deployment settings

# How To

## Enable maven repo on an ubuntu server


This will allow you to present a maven repo with apache2 and allow to deploy maven artifacts there through ftp.

This should be done through ssh

### Create maven user

create the user

```
sudo adduser maven --comment
```

Then change its dir

```
sudo -u maven mkdir -p /home/maven/public_html/release
sudo -u maven mkdir -p /home/maven/public_html/snapshot
sudo chmod -R o+x /home/maven
```


### Install and configure Web server

```
sudo apt install apache2 -y
sudo a2enmod userdir
sudo a2enmod ssl
sudo service apache2 restart
```

check if you have access to local maven dir:

```
curl localhost/~maven/
```

should return a page with the two "release" and "snapshot" folders


### Activate https

First create a self-signed certificate in /etc/ssl/

```
sudo openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -batch -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

Then edit the file `/etc/apache2/sites-enabled/000-default.conf` to make it start with :

```
<VirtualHost *:443>
        SSLEngine on
        SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key    
        SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt    
```

and restart apache2 with `sudo service apache2 restart`

The command `curl https://localhost/~maven/ -k` should return a valid page (-k is because the certificate is self signed)

### Install and configure FTP server

```
sudo apt install proftpd
```

Then in the config file `/etc/proftpd/proftpd.conf` 

1. uncomment the line 

> RequireValidShell off

2. jail your users to the public_html

> DefaultRoot ~/public_html


you then need to restart the server, and connect to your ftp server using the ftp command from your personal computer.

### Configuration in your pom

First add the distributionManagement in the same pom to specify where to deploy from. For an example see that project's pom

You need to add in your `~/.m2/settings.xml` the login information. In my case I names my two repositories kimsufi-ftp-release and kimsufi-ftp-snapshot anbd I have password "mypassword"  (change accordingly) . The settings is not ended because you should have other things.

```
<settings>
  <servers>
    <server>
      <id>kimsufi-ftp-release</id>
      <username>maven</username>
      <password>mypassword</password>
    </server>
    <server>
      <id>kimsufi-ftp-snapshot</id>
      <username>maven</username>
      <password>mypassword</password>
    </server>
  </servers>
```

Then for a project to use your repo, see the project/repositories in that project's pom.
