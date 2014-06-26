This repo contains everything needed to setup an interface for uploading ODB2 data logged from your car in real-time using the [Torque Pro](https://play.google.com/store/apps/details?id=org.prowl.torque) app for Android.

The interface allows the user to:

  * View Google Maps of GPS data from Torque
  * Create time series plots of OBD2 data
  * Easily export data to CSV or JSON

# Screenshots #

Here are some screenshots showing what the interface looks like on both a desktop and mobile device.

### *Desktop* ###
![Full Desktop View](http://storage.googleapis.com/torque_github/examples/open_torque_viewer_0.png)

### *Mobile* ###
<div align="center" style="padding-bottom:15px;"><a href="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_1.png" target="_blank"><img src="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_1.png" width="45%" align="left"></img></a><a href="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_2.png" target="_blank"><img src="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_2.png" width="45%" align="right"></img></a></div>

<br><br>

<div align="center" style="padding-bottom:15px;"><a href="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_5.png" target="_blank"><img src="https://storage.googleapis.com/torque_github/examples/open_torque_viewer_5.png" width="100%" align="center"></img></a></div>

# Server Setup #

### Requirements ###

These instructions assume you already have a LAMP-like server or have access to one. Specifically, you'll need the following:

  * MySQL database
  * Apache webserver
  * PHP server-side scripting

### Create Empty MySQL Database & Configure User ###

First we'll create an empty database that will be configured further in the next section. Once we have an empty database, we'll then create a MySQL user and provide them with the necessary permissions on the database.

Start by opening a MySQL shell as the root user. Then create a database named `torque` and create a user with permission to insert and read data from the database. In this tutorial, we'll create a user `steve` with password `zissou` that has access to all tables in the database `torque` from `localhost`:

```sql
CREATE DATABASE torque;
CREATE USER 'steve'@'localhost' IDENTIFIED BY 'zissou';
GRANT USAGE, FILE TO 'steve'@'localhost';
GRANT ALL PRIVILEGES ON torque.* TO 'steve'@'localhost';
FLUSH PRIVILEGES;
```

### Create MySQL Options File ###

An [options file](https://dev.mysql.com/doc/refman/5.5/en/option-files.html) is a MySQL configuration file that allows you to login to the database without typing out the username and password each time. We'll also set the permissions on this file so that it is just as secure as typing in your MySQL user/password manually each time.

Create a file in your home directory called `.my.cnf` (e.g. */home/myuser/.my.cnf*) and enter the following text into it, replacing the user/password with the one you created:

```
[client]
user="steve"
password="zissou"
```

To protect the contents of this file, set the permissions on it so only the owner of the file (i.e. your system user) can read it and write to it:

```bash
chmod 600 ~/.my.cnf
```


### Create MySQL Table ###


Next we'll create a table in the database to store the raw log data sent from Torque. I've provided a shell script in this repo that will do this for you. Open a terminal in the folder where you cloned this repo and, assuming you put your MySQL options file in your home directory, simply run:

```
git clone https://github.com/econpy/torque
cd torque
mysql < scripts/create_torque_log_table.sql
```


### Configure Webserver ###


At this point, the MySQL settings are all configured. The only thing left to do related to the database is to add your MySQL user/password to the PHP script.

First rename the `creds-sample.php` file to `creds.php`:

```bash
mv web/creds-sample.php web/creds.php
```

then edit your MySQL user and password in the **$db_user** and **$db_pass** fields:

```php
...
$db_host = "localhost";
$db_user = "steve";
$db_pass = "zissou";
$db_name = "torque";
$db_table = "raw_logs";
...
```

Now move the contents of the `web` folder in this repo to your webserver and set the appropriate permissions. Assuming the document root for your Apache server is located at `/var/www`, you could do:

```bash
mv ./web /var/www/torque
chmod -R 644 /var/www/torque/*
chmod 755 /var/www/torque/
```


# Settings in Torque App #

To use your database/server with Torque, open the app on your phone and navigate to:

```
Settings -> Data Logging & Upload -> Webserver URL
```

Enter the URL to your **upload_data.php** script and press `OK`. Test that it works by clicking `Test settings` and you should see a success message like the image on the right:

<div align="center" style="padding-bottom:15px;"><a href="https://storage.googleapis.com/torque_github/torque_webserver_url.png" target="_blank"><img src="https://storage.googleapis.com/torque_github/torque_webserver_url.png" width="49%" align="left"></img></a><a href="https://storage.googleapis.com/torque_github/torque_test_passed.png" target="_blank"><img src="https://storage.googleapis.com/torque_github/torque_test_passed.png" width="49%" align="right"></img></a></div>

The final thing you'll want to do before going for a drive is to check the appropriate boxes on the `Data Logging & Upload` page under the `REALTIME WEB UPLOAD` section. Personally, I have both **Upload to webserver** and **Only when ODB connected** checked.

At this point, you should be all setup. The next time you connect to Torque in your car, data will begin syncing into your MySQL database in real-time!
