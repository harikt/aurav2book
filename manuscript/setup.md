# Setting up your virtual host 

## Apache

We are going to point the virtual host to `aura.localhost`.
If you are in a debain based OS, you want to create a file 
`/etc/apache2/sites-available/aura2.localhost.conf` with the below contents.

```bash
<VirtualHost *:80>
    ServerName aura2.localhost
    ServerAlias www.aura2.localhost
    DocumentRoot /path/to/project/web
    <Directory /path/to/project/web>
        DirectoryIndex index.php
        AllowOverride All
    </directory>
</VirtualHost>
```

`path/to/project` is where you installed the `aura/web-project` or 
`aura/framework-project`.

**NOTE:** Apache 2.4 users might have to add `Require all granted` below `AllowOverride all` in order to prevent a 401 response caused by [the changes in access control](https://httpd.apache.org/docs/2.4/upgrading.html#access).

Enable the site using 

```bash
a2ensite aura2.localhost
```

and reload the apache

```bash
service apache2 reload
```

Before we go and check in browser add one more line in the `/etc/hosts` 

```bash
127.0.0.1	aura2.localhost www.aura2.localhost
```

Done!

Check http://aura2.localhost in your favourite browser.
