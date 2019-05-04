# How to build the HCR Development Environment

This document will explain how to build a development environment to run all web services in HCR on the developers local machine. During the PoC stage, the public service also use the same approach. The EC2 instance was considered as a developer instance. 

## Prerequisites
* MAC OS
* Docker & docker compose installed
* Logged in to docker hub to pull from public repos. HCR login is user: hcloudreports pass: k!NGk0ng
* Git installed 
* Git configured for aws CodeCommit
* Root enabled and the password to login as root
* Port 80 is available and not used by any other service
* Backup of your existing host file in /etc/hosts

## REPO
* GitLab is used as the git repo. 
* Make sure the git configuration is done before moving forward. This is explained in another document. Search for GitLab in google drive. 

## STEPS
	CLONE DOCKER-COMPOSE REPO
In our home folder create a folder named microservices and clone the report from gitlab.
Commands: 
`<mkdir microservices && git clone git@gitlab.com:hcloudreports/docker-compose.git>` 

**Notes:**
The .gitignore file will have the source file folders and the sql backup folders. You have to clone all the source codes to the relevant folders and also download  .sql backups. Make sure you manage the .gitignore files to avoid any crap being copied to git.

	DOWNLOAD THE SOURCE CODE TO .SOURCE FOLDERS
If you are running it for the first time, use the getsource.sh script to clone the source code from all the repos.
`<./gitsource.sh>`
If for some reason it does not clone all the reports, go to each relevant folder and perform a git clone.
Make sure all the source code in all the folders are available before you go to the next step 
    UPDATE THE HOSTS FILE SO THAT ALL THE PRODUCTION URL’S ARE AVAILABLE LOCALLY
Copy the hosts file from the scripts folder so that all the HCR FQDN’s will point to localhost. 
`<sudo cp scripts/hcrhosts /etc/hosts>` 
To switch back the URL’s to the live site
`<sudo cp scripts/hosts /etc/hosts>`
If you already have a hosts file, copy that file to scripts so that you can use the correct host file when you are off development
    TAKE A BACKUP OF THE PRODUCTION DATABASE
Take a backup of the production database to be restored to your local database.
In the docker-compose file mysqlrestore service, make sure *command: ./backup.sh* is not commented, environment variables is pointing to the correct DB you want to backup and all other variable vaules are correct .
Type `<docker-compose up -d mysqlrestore>`
This will start the restore container, connect to the DB, take a backup to the mysql/sqlfiles location and exit
If it did not backup all the db’s, most likely it timed out.  run the following command to backup again.
`<docker-compose exec mysqlrestore ./backup.sh>`
If the container is not running, in the docker-compose file, mysqlrestore service, make sure tty: true is not commented. If it is commented, do a `<docker-compose up -d mysqlrestore>` and `<docker-compose exec mysqlrestore ./backup.sh>` again
Make sure all the .sql files relevant to the DB’s exist in /sql/sqlfiles before you progress to the next section
Once the backup is successful, comment and uncomment  the following in lines in docker-compose.

*mysqlrestore service*
* #command: ./backup.sh
* #tty: true
*mysql service
* - ./mysql/sqlfiles:/home
* - ./mysql/scripts:/docker-entrypoint-initdb.d


	START ALL THE CONTAINERS
When you are running `<docker-compose up -d>` for the first time, make sure the following is not commented in the mysql service.

* - ./mysql/sqlfiles:/home
* - ./mysql/scripts:/docker-entrypoint-initdb.d

This is needed for mysql to create the DB, users and import the database from the previous backup. After first use, make sure these lines are commented in the docker compose. Now run `<docker-compose up -d>`

This will create the whole environment on your local machine

	CHECK IF THE DATABASE IS GOOD
Open a browser and go to http://sqladmin.hcloudreports.com. The server is mysql, username and password is root. Since this DB is only accessible in your local machine, there is no need to change the password.
Check the databases, it’s content and users if they are available. 
**UPDATE THE DOCKER-COMPOSE FILE SO THAT THE DB WILL NOT BE BACKED UP AND RESTORED AGAIN**

Comment the following in the docker-compose file

Mysql service
* - ./mysql/sqlfiles:/home
* - ./mysql/scripts:/docker-entrypoint-initdb.d
 


	MYSQL BACKUP

command: `<./backup.sh>`
Save the docker-compose.yml file.
Do not perform a get push origin master. Only the the ops engineer is allowed to edit this repo.
Docker ps and docker-compose ps to see if all containers are running.

	HOW TO TAKE A BACKUP OF YOUR LOCAL DB TO S3
You can take a backup of your local database to S3. This is performed using the mysqlbackups service. In the service environment section, change the value of the following variable. This is quite important to identify your specific backup in S3.

*BACKUP_SOURCE=aws-test-server-dockercompose*

The value could be like *dinesh-macbook-dockercompose*

Save the docker-compose file and run `<docker-compose up -d>`. This will update the container to it’s new state. Type the following command to make sure the environment variable has changed. `<docker-compose exec mysqlbackup env>'

If the variable has not change, run `<docker-compose restart mysqlbackups>`

To take a backup type the following command
`<cd mysqlbackups>`
`<./baskup.sh>`

To see the S3 folder to ensure your backup exist, type the following command

`<docker-compose exec mysqlbackups aws s3 ls s3://hcr-db-backups>`



