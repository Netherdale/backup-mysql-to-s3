# Backup a MySQL database to Amazon S3 using shell script

1. Install AWS CLI

We need to ensure the system installed the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

```sh
$ aws --version
aws-cli/1.17.0 Python/2.7.16 Linux/4.x-amd64 botocore/1.14.0
```

2. Backup MySQL database to S3 (Shell script)

Below is a shell script to dump the database with `mysqldump` and `gzip` it into a folder, later uses the `aws` command to upload the `gzip` file to Amazon S3.

```sh
#!/bin/bash

################################################################
##
##   MySQL Database Backup To Amazon S3
##   Written By: YONG MOOK KIM
##   https://www.mkyong.com/mysql/mysql-backup-and-restore-a-database-or-table/
##   https://www.mkyong.com/linux/how-to-zip-unzip-tar-in-unix-linux/
##
##   Daily backup, At 01:30
##   30 1 * * * /home/ubuntu/scripts/backup-mysql.sh > /dev/null 2>&1

################################################################

NOW=$(date +"%Y-%m-%d")
NOW_TIME=$(date +"%Y-%m-%d %T %p")
NOW_MONTH=$(date +"%Y-%m")

MYSQL_HOST="localhost"
MYSQL_PORT="3306"

MYSQL_DATABASE="wordpress"
MYSQL_USER="user"
MYSQL_PASSWORD="password"

BACKUP_DIR="/home/mkyong/backup/$NOW_MONTH"
BACKUP_FULL_PATH="$BACKUP_DIR/$MYSQL_DATABASE-$NOW.sql.gz"

AMAZON_S3_BUCKET="s3://mkyong/backup/linode/mysql/$NOW_MONTH/"
AMAZON_S3_BIN="/home/mkyong/.local/bin/aws"

#################################################################

mkdir -p ${BACKUP_DIR}

backup_mysql(){
       mysqldump -h ${MYSQL_HOST} \
         -P ${MYSQL_PORT} \
         -u ${MYSQL_USER} \
         -p${MYSQL_PASSWORD} ${MYSQL_DATABASE} | gzip > ${BACKUP_FULL_PATH}
}

upload_s3(){
      ${AMAZON_S3_BIN} s3 cp ${BACKUP_FULL_PATH} ${AMAZON_S3_BUCKET}
}

backup_mysql
upload_s3


# check if any error
#if [ $? -eq 0 ]; then
# ok, no error, log and send email?
#else
# failed, got error, log and send email?
#fi;
```

3. How to run the backup script?

We must assign a `+x` execute permission to run the shell script.

```sh
$ chmod +x backup-mysql.sh

# run the backup
$ ./backup-mysql.sh
```
