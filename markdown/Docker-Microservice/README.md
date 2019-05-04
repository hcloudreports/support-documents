# Docker Compose to run services that is only being developed

> - Author: Elwin Pietersz
> - Date last updated: 08/04/2019
> - Version 0.2



| Version                                            | Description                                                                                                                                                         |
|:---------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0.1                               | 07/04/2018: Initial development
| 0.2                               | 08/04/2018: Introduced submodules
 

## Introduction

The original docker compose file loads the entire HCR platform. The developer(s) would like to load few microservices to develop only the service(s) being deloped. This project will help the deveoper get started. It is expected the developer to have some basic knowledge on git, docker to use this project and make changes. 

<span style="color:red">**Important Note:**</span> Always pull from this repor. **Do not push to update this repo**. It is likely that you will not have permission to push. If you wish to make changes and maintain your own copy, foke this project and maintain your own repo. 

- How to foke a project [foke](https://docs.gitlab.com/ee/gitlab-basics/fork-project.html#foke).

## Git Submodules
Submodules was chosen insted of Subtree.
- Submodules are easier to push but harder to pull – This is because they are pointers to the original repository
- Subtrees are easier to pull but harder to push – This is because they are copies of the original repository

For the purpose of this project, once this repo is pulled, the developer should not be making any changes to remote. However the develop can always fork the repo and maintain his/her own project/repo. Fundementally the developer will be working on the submodule repo, e.g. app/developer/sourcecode. Submodules seems to be the best option for this purpose. 

<span style="color:red">**The golden rule of modifying submodules**<span style="color:red">
Always commit and push the submodule changes first, before then committing the submodule change in the parent repository.

A submodule is nothing but a pointer to a specific commit in an external repository, Without following this rule you can get into a confusing state in which the parent repository is pointing to a submodule commit that only exists on your local machine. The tooling should warn about this and reject the push, but I haven’t seen it happen yet.

If you don’t notice that you need to update the submodule, all it takes is a lazy git add -A or git commit -a and you’ve downgraded the submodule to the version you’ve had in your working copy all along. **This stale submodule can cause the entire project to get into a mess**.

## Implementation Details

Create a self-explanatory folder in your local machine e.g. **developer** and clone the git repository using the command:

`mkdir <folder name> && git clone git@gitlab.com:hcr-development/ops/docker-ms.git <folder name>`

You will see the following folders and files in the root folder.

| Folder                                            | Description                                                                                                                                                         |
|:---------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| app                               | The root folder will/should contain folders with the name representing the service, e.g. developer. In this folder there is a dockerfile where you can use to build a docker container with the source code if needed. However this is not used since the docker composer file fill use bind volume mounts. The app/sourcecode folder will have your source code for the service. **Note:** You have to manually create the folder and clone the source codes because this folder is excluded in the .gitignore file. 
| hostfiles                               | There are some hostfiles for you to test once the services are running. Update as per your need. When you are ready to test, copy the localhost file to your /etc/hosts folder. `sudo cp hostfiles/localhost /etc/hosts`
| mysql                               | This folder contains everything you need to run the mysql database. The docker files can be used to build the database with all the databases included in it. This is not used since bind mounts are used in docker compose. Before you start any work prepare the database and make sure that the files are in scripts and sqlfiles folder. sqlfiles folder will not exist when you do get pull. You need to manually create this folder. **scripts:** this folder will have restore script. Update this according to the database you are working on. **sqlfiles:** this folder contain the database to import. If this is s new database you are creating, inlcude only the database create command. Subsequently you can use sqladmin to work on the database. <span style="color:red">**Remember the database is ephemeral. This means when docker stop the database, you lose all the data in it. The sqlfiles and scripts folder will be used to create and load the data to the database every time you start the mysql container.**<span style="color:red">
| proxy                               | The docker file is used to build the proxy with the nginx.conf file built into it. This is not used since bind mounts are used in the docker compose file. The nginx.conf depends on all the services it support to reverse proxy to be up. Hence the depends_on directive in the docker compose file. Make sure this is updated based on the service you are developing. The file consist of an example service where you can copy paste to contruct the service you are developing. 
| README.md                               | You do not need to modify this file. The author will maintan this. If you have any comments to update this file, create a issue in the project and assign it to the author.
| docker-compose.yml                               | You only need to change the service you are developing. rproxy, adminer, mysql should not be changed or deleted. 

**MYSQL LOGIN:**

[http://sqladmin.hcloudreports.com](http://sqladmin.hcloudreports.com) will provide you sqladminer UI to manage the mysql server. <span style="color:red">Make sure your hostfile point this FQDN to local host always to avoid accidental changes to the production database.<span style="color:red">

To login: The **user and password** is always root. This database is internal only. So there is no need to maintain a secure password method. 

**Application using the Database:**
Note the docker-compose file environment directive.

```bash
    environment:
      MYSQL_SERVICE: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DATABASE: system_db
```

This information is passed to the docker container as an environment variable. You application should be using these variables. Sample code for NodeJS and PHP.

```bash
    var  con = mysql.createConnection({
    	host:process.env.MYSQL_SERVICE,
    	user:process.env.MYSQL_USER,
    	password:process.env.MYSQL_PASSWORD,
    	database:process.env.MYSQL_DATABASE

});

$db_host = getenv('MYSQL_SERVICE');
$db_user = getenv('MYSQL_USER');
$db_pass = getenv('MYSQL_PASSWORD');
$db_name = getenv('MYSQL_DATABASE');
```

**Note:** When GitLab CI/CD is deploying and managing the containers, protected environment varialbles will be used. This means the password will not be set via the docker compose file. It will be available to the source code via GitLab. 

GitLab CI/CD environment variables [Link](https://docs.gitlab.com/ee/ci/variables/).



**IMPORTANT TIPS:**

- Always use `docker-compose config` command to validate your docker compose file before starting
- On another window/screen always have the `docker-compose logs -f` running as soon as you invoke `docker-compose up -d`. <span style="color:red">**Since we are using host files. If you failed to update your local host file, you could potentally be making changes to the production service. So it is quite important to know if your requests are pointing to the service you are running. The log will always tell you this for each http request.</span>
- Always use `docker-compose ps` before you run `docker-compose up -d` to ensure there is no other containers running and will not conflict with ports. Note that only port 80 is mapped to your local machine. This is to communicate with the proxy. All other services are internal to the network docker compose create which is named api-network. Stop all containers that could possibly conflict with the containers in the docker compose file before you run docker-compose up.



## Troubleshooting Tips

- If a container filed to start, the first place to check is logs. Following commands can be used to check logs.

```bash
    docker-compose logs -f
    docker-compose logs <service name e.g. rproxy> -f
    docker logs <container id> 
```

- After you start the containers, if you are unable to access a perticuler service, do a  `docker ps` to find out what containers are running. 
- If you want to check if one container can communicate with another container, you can connect to the container and run some commands. E.g. Lets assume the developer container cannot communicate with internal-messaging container and you want to try some curl scripts. To connect to the developer container `docker-compose exec developer /bin/bash`. One you are in the container, you can `curl -vvv internal-messaging`. This will tell you if you can communicate with the API container. You can also do curl commands to so if the API is working. You can also use the docker command to connect to a container. `docker exec -it <container id> /bin/bash`. CTL + C will get you out of the container. 
- If you need to restart a perticuler service use the command `docker-compose restart <service name>` e.g. to restart the proxy, `docker-compose restart rproxy`
- To stop docker compose `docker-compose down`
- To remove containers at stopped state `docker rm $(docker ps -a -q)`
- To list all the docker images `docker images`
- To delete a perticuler image `docker rmi <image name>`
- To delete all the images `docker rmi $(docker images -f)`

## Step-By-Step for the less competent 

```bash
    # Assuming we are wokring on the devleoper service and the internal-messaging API where this was originally built.
    # in your home folder, create a folder named docker-ms
    mkdir docker-ms
    # clone the repo, prepare and run
    git clone git@gitlab.com:hcr-development/ops/docker-ms.git docker-ms
    cd docker-ms
    ls -la
    git status
    git submodule status
    cat .gitmodules
    # Example output
    [submodule "app/internal-messaging/sourcecode/internal-messaging"]
        path = app/internal-messaging/sourcecode/internal-messaging
        url = git@gitlab.com:hcr-development/dev/internal-messaging.git
    [submodule "app/developer/sourcecode/developer"]
        path = app/developer/sourcecode/developer
        url = git@gitlab.com:hcr-development/dev/developer.git
    
    # Pull the submodules from the remote repo    
    git submodule init
    git submodule 
    
    # If you intend to work on a diffrent project, e.g. developer-reportmodules, do the following.
    rm .gitmodules
    cd app
    rm -rf .
    cd ..
    git status
    # output should look like below
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
    (use "git push" to publish your local commits)
    nothing to commit, working tree clean
    
    # below should return empty
    git submodule status
    
    #now you are ready to create a new submodule
    cd app
    git submodule add git@gitlab.com:hcr-development/dev/developer-reportmodules.git
    cd ..
    git add -A
    git commit "you comment"
    # Below commands will tell you that nothing to commit and the status of submodules
    git status
    git submodule status
    cat .git modules
    
    # to prepare your database
    cd mysql
    mkdir sqlfiles
    # copy the backup database to this folder
    cp system_db.sql sqlfiles
    # make note of the restore script in the mysql/scripts folder
    # update your local hostfile
    sudo cp hostfiles/localhost /etc/hosts
    # make sure no containers are running
    docker ps
    docker-comnpose up -d
    docker -ps
     
```

## Command help when working with git submodules

```bash
    #cd to the folder where you have your submodule e.g.
    cd app/internal-messaging/sourcecode/internal-messaging
    git status
    # will provide you a output similer to
    On branch feature2
    Your branch is up-to-date with 'origin/feature2'.
    nothing to commit, working tree clean
    # note that you are in a perticuler branch. If you are not in the correct branch you should switch to it. If you do not have any branches and  you want to work on a brach in remote, fetch and then switch. 
    git fetch
    git branch feature2
    # you can now work on this branch making all your changes. Make sure you add, commit and push before you do the same in the parent branch.     Always do git branch to check where you are. 

```



## Links & References

- Get started with Docker Compose [Getting Started](https://docs.docker.com/compose/gettingstarted/).
- Nginx reverse proxy [nginx proxy](https://dev.to/domysee/setting-up-a-reverse-proxy-with-nginx-and-docker-compose-29jg).
- Subtree and Submodule [Subtree/Submodule](https://codewinsarguments.co/2016/05/01/git-submodules-vs-git-subtrees/).

