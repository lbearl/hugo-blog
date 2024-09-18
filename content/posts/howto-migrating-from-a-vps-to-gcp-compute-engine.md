---
title: "HOWTO: Migrating from a VPS to GCP Compute Engine"
date: "2017-04-13"
categories: 
  - "dev-ops"
  - "public-cloud"
  - "system-administration"
---

This post is going to be a brief guide on how to migrate a wordpress site from an existing host to Google Cloud's Compute Engine service. Note that this guide assumes that multiple sites are being migrated, if only one is then that should make things slightly simpler.

At the end of this guide the site will be migrated, an SSL certificate from [Let's Encrypt](https://letsencrypt.org/) will be provisioned and Apache will be doing it's thing. I'll leave it as an exercise to the reader to put the site behind CloudFlare (hint: if you are already there the only thing you probably have to change is your A records). For my purposes this guide will also include migrating all images and attachments to Google Cloud Storage.

## Step 1: Back up EVERYTHING

In order to make this all work you are going to need backups of everything: the databases, the existing wordpress installs, any other assets you are using, etc. The easy way of doing this is to ssh to your web server and just `tar cvzf my_site.tgz /var/www/my_site` (this creates a tarball). Then use something like `sftp` or `scp` in order to copy the tarball to your local machine. Rinse and repeat for each site.

Repeat this for the database server. Assuming you have 1 database per site (my preferred way of doing it) you can just `mysqldump -u root -p <pw> mydb > mydb.sql`. If your database is huge it might be prudent to zip or otherwise compress the dump file (mine are only a few megabytes, so I didn't bother). Once you have the dump file, copy it to your local machine. Keep it up until you have all your databases.

## Step 2: Make the Cloud a thing

Now that you have all of the goods on your local machine, it's time to make a place for them to live. To that end you'll want to provision a brand new VM on the Compute Engine. I'm not going to walk you through that process as it is pretty well documented (and also just consists of pressing a few buttons. The one thing to watch out for is to make sure that the VM has _all_ of the API Access Scopes that you will need as in order to change them you first have to power off the virtual machine (which is stupid and lots of people have complained about, but that is how it is). ![](images/api-access-scopes.png)

While that is booting up, let's get the MySQL stuff setup.

If you're moving to Google Cloud, you might as well move in fully so for MySQL we'll use Google's SQL - MySQL Second Generation. The only real weird part here is to make sure you whitelist the IP address of the VM you setup. Alternatively you can go through a process to use the Google SQL Cloud Proxy, but IP whitelisting is a lot easier (and has fewer moving parts).

## Step 3: To the CLOUD

Now that your data has a place to live, it's time to start pushing those bytes to google. This is a 2 step process: 1. Upload the wordpress tarballs to the vm (assuming it has finished booting by now). The easiest way I've found to do this is to install [gcloud](https://cloud.google.com/sdk/) and then execute `gcloud compute copy-files ~/local/path <vm_name>:~/` replacing the chevrons and vm\_name with the name of the instance you created (mine is web1 because I'm all sorts of imaginative when it comes to naming servers). 2. Navigate to Google Cloud Storage and create a new bucket which is _not_ publicly accessible. After the bucket is created upload all of the sql dumps (in an uncompressed form).

## Step 4: Make it Live!

All of the parts are in place, they just need to be configured properly and you'll have all your blogs running.

### Web Server

Decompress the tarballs so that you have the wordpress directories again: `tar xvzf my_site.tgz`. Copy the resultant directory to wherever you want your blog to live (I just throw them in `/var/www/`). Next up you'll need to setup Apache to know about your site.

Since we are only going to support SSL, we will only configure virtual hosts files for SSL. Your configuration should look something like this

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin admin@example.org
                ServerName lukebearl.com
                ServerAlias www.lukebearl.com

                DocumentRoot /path/to/lukebearl.com

                <Directory />
                        Options FollowSymLinks
                        AllowOverride None
                </Directory>

                <Directory /path/to/lukebearl.com>
                        Options Indexes FollowSymLinks MultiViews
                        AllowOverride All
                        Require all granted
                </Directory>

                # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
                # error, crit, alert, emerg.
                # It is also possible to configure the loglevel for particular
                # modules, e.g.
                #LogLevel info ssl:warn

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                # For most configuration files from conf-available/, which are
                # enabled or disabled at a global level, it is possible to
                # include a line for only one particular virtual host. For example the
                # following line enables the CGI configuration for this host only
                # after it has been globally disabled with "a2disconf".
                #Include conf-available/serve-cgi-bin.conf

                #   SSL Engine Switch:
                #   Enable/Disable SSL for this virtual host.
                SSLEngine on

                #   A self-signed (snakeoil) certificate can be created by installing
                #   the ssl-cert package. See
                #   /usr/share/doc/apache2/README.Debian.gz for more info.
                #   If both key and certificate are stored in the same file, only the
                #   SSLCertificateFile directive is needed.
                SSLCertificateFile    /etc/letsencrypt/live/lukebearl.com/cert.pem
                SSLCertificateKeyFile /etc/letsencrypt/live/lukebearl.com/privkey.pem
                SSLCertificateChainFile /etc/letsencrypt/live/lukebearl.com/fullchain.pem

                # ... <snipsnip> ...

        </VirtualHost>
</IfModule>
```

Once you have that setup execute `sudo service apache2 reload` and then setup the Let's Encrypt certificate.

In order to do that you'll first need to make sure that you have certbot installed. After that run this command: `sudo certbot certonly --webroot --webroot-path /var/www/html/ --renew-by-default --email youremail@example.org --text --agree-tos -d lukebearl.com -d www.lukebearl.com` The /var/www/html is still serving the default document on port 80 due to Apache's default site (which we never disabled).

You'll also want to make sure that the renewal is scheduled in a crontab entry.

At this point you should be able to navigate to your site (and get a database connection error).

### SQL

From my SQL control panel in console.cloud.google.com, click the "import" button and then in the dialog that appears find each of the dump files in turn. You'll also want to use the SQL Console in order to create a user with rights to all of those wordpress databases, but who isn't root. Make sure the password is decently strong.

After you are done importing all of the databases, go back to the web server and the final configuration task before your site is live can be done.

### Editing wp-config.php

`cd` into the wordpress directory and then open the wp-config.php file in your text editor of choice (like vim). You'll need to look for and edit all of the DB\_\* settings to reflect your new MySQL instance. Pay attention to the DB\_HOST as that should be the IPv4 address from the SQL management pane.

## Extra Credit: Images

If you run an image heavy blog (which this blog obviously is an example of), you'll notice a considerable speed-up if you make use of the [Google Cloud Storage wordpress plugin](https://wordpress.org/plugins/gcs/). One big gotcha that I [found](https://github.com/GoogleCloudPlatform/wordpress-plugins/issues/42) is that php5-curl _must_ be installed or this plugin breaks. A big thanks to the developers who work on that plugin as they quickly helped resolve the issue.

Moving the images is a kind of 2 step process: 1. Create the bucket (and make sure to assign the allUsers user "Read" access before you upload anything) - You probably want to create the bucket as "Multi-Regional" so that images get cached to edge locations. - Also create a new folder in the bucket named "1". 2. Copy all of the files from the wp-content/uploads directory to the bucket. When doing this I have found it easiest to `cd` into the `wp-content/uploads` directory and then execute the following: `gsutil -m cp -r . gs://<bucket_name>/1/` - This may take a few minutes to complete. The `-m` flag will make it multithreaded. 3. Login to the Wordpress Admin and install the plugin. - After installing, configure the plugin to point to the bucket you just copied everything into. Also make sure that the "Use Secure URLs for serving" flag is checked or you'll end up with mixed-content errors.

At this point you should have your blog setup on Google Compute Engine using the Cloud MySQL instance and all of your images should be hosted through Google Storage. Let me know how it goes! [@lukebearl](http://www.twitter.com/lukebearl)
