How to update Ghost

Link to bitnami ghost docs: https://github.com/bitnami/bitnami-docker-ghost#upgrade-this-application
Link to ghost docs: https://ghost.org/docs/update/

Step I

Backup the content by going to the admin area (/ghost) -> Labs -> Export your content.
Now backup the ghost persistance content:
    
    cd /var/docker/ghost/ghost-persistence/ghost/ && tar cfvz ~/content_backup.tar.gz content
    
To check the content of the tar use

     tar -ztvf ~/content_backup.tar.gz
     
You can also transfer the snapshot via scp to your machine, current directory:

    scp hal:content_backup.tar.gz .
    

Let's follow bitnami upgrade steps:

    @hal:~$ docker pull bitnami/ghost:latest
    @hal:/var/docker/ghost$ docker-compose stop blog
    @hal:/var/docker/ghost$ sudo rsync -a /var/docker/ghost/ghost-persistence /var/docker/ghost/ghost-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
    
    @hal:~$ docker pull bitnami/mariadb:latest
    @hal:/var/docker/ghost$ docker-compose stop mariadb
    @hal:/var/docker/ghost$ sudo rsync -a /var/docker/ghost/mariadb-persistence /var/docker/ghost/mariadb-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
    
    @hal:/var/docker/ghost$ docker-compose rm -v mariadb
        Going to remove ghost_mariadb_1
        Are you sure? [yN] y
        Removing ghost_mariadb_1 ... done
    
    @hal:/var/docker/ghost$ docker-compose rm blog
        Going to remove ghost_blog_1
        Are you sure? [yN] y
        Removing ghost_blog_1 ... done
    
    @hal:/var/docker/ghost$ docker-compose up
    
    
    
    
    
        
    
