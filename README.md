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
# actually not used yet
sudo a2enmod ssl
sudo service apache2 restart
```

check if you have access to local maven dir:

```
curl localhost/~maven/
```

should return  a page with the two "release" and "snapshot" folders

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

First add your repository to fetch the artifacts. See the project/repositories in that project's pom.

Then add the distributionManagement in the same pom to specify where to deploy from. Again, see that project's pom

Finally add in your `~/.m2/settings.xml` the login information. In my case I names my two repositories kimsufi-ftp-release and kimsufi-ftp-snapshot anbd I have password "mypassword"  (change accordingly) . The settings is not ended because you should have other things.

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

