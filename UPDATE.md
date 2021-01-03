### How to update Ghost

Link to bitnami ghost docs: https://github.com/bitnami/bitnami-docker-ghost#upgrade-this-application<br/>
Link to ghost docs: https://ghost.org/docs/update/

Step I

Backup the content by going to the admin area (/ghost) -> Labs -> Export your content.
Now backup the ghost persistance content:
    
    cd /var/docker/ghost/ghost-persistence/ghost/ && tar cfvz ~/content_backup.tar.gz content
    
To check the content of the tar use

     tar -ztvf ~/content_backup.tar.gz
     
You can also transfer the snapshot via scp to your machine, current directory:

    scp hal:content_backup.tar.gz .
    

Step II

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
    
### Following problems...

1)

    blog_1     | WARN  ==> You set the environment variable ALLOW_EMPTY_PASSWORD=yes. For safety reasons, do not use this flag in a production environment.

2)

    blog_1     | mysql-c INFO  Trying to connect to MySQL server
    mariadb_1  | 2021-01-03 20:29:46 3 [Warning] Aborted connection 3 to db: 'unconnected' user: 'unauthenticated' host: '172.31.0.4' (This connection closed normally without authentication)
    blog_1     | mysql-c INFO  Found MySQL server listening at mariadb:3306
    blog_1     | mysql-c INFO  MySQL server listening and working at mariadb:3306
    
3)

    blog_1     | [20:29:50] Ensuring user is not logged in as ghost user [started]
    blog_1     | [20:29:50] Ensuring user is not logged in as ghost user [failed]
    blog_1     | [20:29:50] → You can't run commands with the "ghost" user. Switch to your own user and try again.
    
4)

    blog_1     | [20:29:50] Checking if logged in user is directory owner [started]
    blog_1     | Your user does not own the directory /opt/bitnami/ghost. This might cause permission issues.
    blog_1     | [20:29:50] Checking if logged in user is directory owner [completed]
    
5)

    blog_1     | [20:29:50] Checking folder permissions [started]
    blog_1     | [20:29:52] Checking folder permissions [failed]
    blog_1     | [20:29:52] → Your installation folder contains some directories or files with incorrect permissions:
    blog_1     | - ./
    blog_1     | - ./versions
    blog_1     | - ./versions/3.40.2
    blog_1     | - ./versions/3.40.2/content
    blog_1     | - ./versions/3.40.2/content/data
    ...
    blog_1     | - ./versions/3.40.2/core/server/models/base
    blog_1     | - ./licenses
    blog_1     | - ./logs
    blog_1     | Run sudo find ./ -type d -exec chmod 00775 {} \; and try again.
    
6)

    blog_1     | [20:29:52] Checking file permissions [started]
    blog_1     | [20:29:52] Checking file permissions [failed]
    blog_1     | [20:29:52] → Your installation folder contains some directories or files with incorrect permissions:
    blog_1     | - ./.ghost-cli
    blog_1     | - ./licenses/ghost-3.40.2.txt
    blog_1     | Run sudo find ./ ! -path "./versions/*" -type f -exec chmod 664 {} \; and try again.
    
7)

    blog_1     | One or more errors occurred.
    blog_1     |
    blog_1     | 1) Ensuring user is not logged in as ghost user
    blog_1     |
    blog_1     | Message: You can't run commands with the "ghost" user. Switch to your own user and try again.
    blog_1     | Help: https://ghost.org/docs/install/ubuntu/#create-a-new-user-
    blog_1     |
    blog_1     |
    blog_1     | 2) Checking folder permissions
    blog_1     |
    blog_1     | Message: Your installation folder contains some directories or files with incorrect permissions:
    blog_1     | - ./
    blog_1     | - ./versions
    ...
    blog_1     | - ./versionsDebug Information:
    blog_1     |     OS: Debian GNU/Linux, v10
    blog_1     |     Node Version: v14.15.3
    blog_1     |     Ghost Version: 3.40.2
    blog_1     |     Ghost-CLI Version: 1.15.3
    blog_1     |     Environment: production
    blog_1     |     Command: 'ghost start'
    blog_1     |
    blog_1     | Try running ghost doctor to check your system for known issues.
    blog_1     |
    blog_1     | You can always refer to https://ghost.org/docs/api/ghost-cli/ for troubleshooting.
    ghost_blog_1 exited with code 1
    
    
