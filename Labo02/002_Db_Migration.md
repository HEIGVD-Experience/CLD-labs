# Database migration

In this task you will migrate the Drupal database to the new RDS database instance.

![Schema](./img/CLD_AWS_INFA.PNG)

## Task 01 - Securing current Drupal data

### [Get Bitnami MariaDb user's password](https://docs.bitnami.com/aws/faq/get-started/find-credentials/)

```bash
[INPUT]
cat /home/bitnami/bitnami_credentials | grep "default username and password is"

[OUTPUT]
The default username and password is 'not the username' and 'not the password'.
```

### Get Database Name of Drupal

```bash
[INPUT]
mariadb -u root -e 'SHOW DATABASES' -p

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
```

### [Dump Drupal DataBases](https://mariadb.com/kb/en/mariadb-dump/)

```bash
[INPUT]
mariadb-dump -u root -r dump.sql bitnami_drupal -p

[OUTPUT]
There is no response with this commmand.
```

### Create the new Data base on RDS

```sql
[INPUT]
mariadb --host dbi-devopsteam09.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u admin -p
CREATE DATABASE bitnami_drupal;
```

### [Import dump in RDS db-instance](https://mariadb.com/kb/en/restoring-data-from-dump-files/)

Note : you can do this from the Drupal Instance. Do not forget to set the "-h" parameter.

```sql
[INPUT]
mariadb --user admin --host dbi-devopsteam09.cshki92s4w5p.eu-west-3.rds.amazonaws.com --password bitnami_drupal < dump.sql

[OUTPUT]
There is no response with this commmand but you can check if the db is correctly imported by connecting to MariaDB and running :
use bitnami_drupal
show tables;
```

### [Get the current Drupal connection string parameters](https://www.drupal.org/docs/8/api/database-api/database-configuration)

```bash
[INPUT]
tail -n 15 /opt/bitnami/drupal/sites/default/settings.php

[OUTPUT]
#   include $app_root . '/' . $site_path . '/settings.local.php';
# }
$databases['default']['default'] = array (
  'database' => 'bitnami_drupal',
  'username' => 'bn_drupal',
  'password' => 'not a password',
  'prefix' => '',
  'host' => '127.0.0.1',
  'port' => '3306',
  'isolation_level' => 'READ COMMITTED',
  'driver' => 'mysql',
  'namespace' => 'Drupal\\mysql\\Driver\\Database\\mysql',
  'autoload' => 'core/modules/mysql/src/Driver/Database/mysql/',
);
$settings['config_sync_directory'] = 'sites/default/files/config_NfdayREqv22vTYlQztq93slHghceWLXdoYq3fvTiOWanWWKvNw9-jhJhobONgHDhitEqI1ElnA/sync';
```

### Replace the current host with the RDS FQDN

```
//settings.php

sudo vim /opt/bitnami/drupal/sites/default/settings.php

$databases['default']['default'] = array (
   [...] 
  'host' => 'dbi-devopsteam09.cshki92s4w5p.eu-west-3.rds.amazonaws.com',
   [...] 
);
```

### [Create the Drupal Users on RDS Data base](https://mariadb.com/kb/en/create-user/)

Note : only calls from both private subnets must be approved.
* [By Password](https://mariadb.com/kb/en/create-user/#identified-by-password)
* [Account Name](https://mariadb.com/kb/en/create-user/#account-names)
* [Network Mask](https://cric.grenoble.cnrs.fr/Administrateurs/Outils/CalculMasque/)

```sql
[INPUT]
CREATE USER bn_drupal@'10.0.9.%' IDENTIFIED BY 'not a password';

GRANT ALL PRIVILEGES ON bitnami_drupal.* TO bn_drupal@'10.0.9.%';

FLUSH PRIVILEGES;
```

```sql
//validation
[INPUT]
SHOW GRANTS for 'bn_drupal'@'10.0.9.%';

[OUTPUT]
+-----------------------------------------------------------------------------------------------------------------+
| Grants for bn_drupal@10.0.9.%                                                                                   |
+-----------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bn_drupal`@`10.0.9.%` IDENTIFIED BY PASSWORD 'not a password' |
| GRANT ALL PRIVILEGES ON `bitnami_drupal`.* TO `bn_drupal`@`10.0.9.%`                                            |
+-----------------------------------------------------------------------------------------------------------------+
```

### Validate access (on the drupal instance)

```sql
[INPUT]
mariadb -h dbi-devopsteam09.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u bn_drupal -p

[INPUT]
show databases;

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)
```

* Repeat the procedure to enable the instance on subnet 2 to also talk to your RDS instance.
