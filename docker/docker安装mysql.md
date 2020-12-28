# docker 安装mysql

- 更改国内镜像
  ` /etc/docker/daemon.json`，

  ```
  {
      "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
  }
  ```

  

- 使用`docker search mysql`，可以搜索mysql的版本。
  
  ```bash
  # root @ localhost in ~/playground/docker [1:06:14] C:1
  $ docker search mysql
  NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
  mysql                             MySQL is a widely used, open-source relation…   9775                [OK]
  mariadb                           MariaDB is a community-developed fork of MyS…   3566                [OK]
  mysql/mysql-server                Optimized MySQL Server Docker images. Create…   717                                     [OK]
  centos/mysql-57-centos7           MySQL 5.7 SQL database server                   78
  mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   73
  centurylink/mysql                 Image containing mysql. Optimized to be link…   61                                      [OK]
  bitnami/mysql                     Bitnami MySQL Docker Image                      44                                      [OK]
  deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                                      [OK]
  tutum/mysql                       Base docker image to run a MySQL database se…   35
  schickling/mysql-backup-s3        Backup MySQL to S3 (supports periodic backup…   30                                      [OK]
  prom/mysqld-exporter                                                              29                                      [OK]
  databack/mysql-backup             Back up mysql databases to... anywhere!         28
  linuxserver/mysql                 A Mysql container, brought to you by LinuxSe…   25
  centos/mysql-56-centos7           MySQL 5.6 SQL database server                   19
  circleci/mysql                    MySQL is a widely used, open-source relation…   19
  mysql/mysql-router                MySQL Router provides transparent routing be…   16
  arey/mysql-client                 Run a MySQL client from a docker container      14                                      [OK]
  fradelg/mysql-cron-backup         MySQL/MariaDB database backup using cron tas…   8                                       [OK]
  openshift/mysql-55-centos7        DEPRECATED: A Centos7 based MySQL v5.5 image…   6
  genschsa/mysql-employees          MySQL Employee Sample Database                  5                                       [OK]
  devilbox/mysql                    Retagged MySQL, MariaDB and PerconaDB offici…   3
  ansibleplaybookbundle/mysql-apb   An APB which deploys RHSCL MySQL                2                                       [OK]
  jelastic/mysql                    An image of the MySQL database server mainta…   1
  widdpim/mysql-client              Dockerized MySQL Client (5.7) including Curl…   1                                       [OK]
  monasca/mysql-init                A minimal decoupled init container for mysql    0
  
  # root @ localhost in ~/playground/docker [1:07:26]
  $
  ```
  
-   `docker pull mysql/mysql-server:tag`，拉取特定tag的mysql版本镜像。比如拉取mysql 8.0，你可以`docker pull mysql/mysql-server:8.0.21-1.1.17`，拉取mysql 5.6 你可以`docker pull mysql/mysql-server:5.6.49`，至于说后面的tag去哪里查，可以去官方repository看。这里，https://hub.docker.com/r/mysql/mysql-server/tags/。（这个似乎不是官方？？），或者这里才是官方？https://hub.docker.com/_/mysql?tab=tags

