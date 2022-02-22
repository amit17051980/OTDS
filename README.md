# OpenText™ Directory Services (OTDS)

## What is OpenText Directory Services?
If you are familiar with IdaaS (Identity as a Service) concept, and have used OKTA/Auth0/Azure-AD, this is quite similar, but it has some main differences:
  1. This is infrastructure independent. i.e. run in local PC or cloud instances or Kubernentes clusters on any Winodws/Unix. 
  2. This provides hybrid approach to replicate users from various sources (AD, LDAP, etc.)
  3. It provides great features to documentum eco system.

Directory Services is a repository of user and group identity information and a collection of services to manage this information for OpenText applications. OTDS contains components for identity synchronization and single sign on for all OpenText applications. Directory Services offers synchronization and authentication features that can help your organization save time, and administrative overhead, by enabling you to maintain user information in one directory, for use by multiple OpenText applications. For example, you can base your OpenText Content Server user information on the user information already contained in your Windows domain. If your organization maintains several Enterprise Server systems, they can all use the same central user directory. Directory Services can synchronize with your identity provider to pull user and group information from your identity provider automatically. Directory Services then pushes these users and groups to your OpenText applications automatically and incrementally. This synchronization of user and group data across OpenText applications allows Directory Services to enable single sign on and secure access to all OpenText applications.

![Alt text](https://github.com/amit17051980/OTDS/blob/main/Architecture-1.png "Figure-1")

![Alt text](https://github.com/amit17051980/OTDS/blob/main/Architecture-2.png "Figure-2")

## Installation Steps (Manual Docker Build)
Although official guides are there to use Helm Charts on Kubernetes Cluster, but I find it easy to start with basic docker containers. This allow me to understand the basis installation procedure and at the same time using containers to re-provision things fast in case things are not going well.

Here are some instructions that could help you provision OTDS for your POC. In my use case I'm using Postgres as Database, Unix as OS and Tomcat 10 with OpenJDK 11.

  1. Provision Postgres Docker Container
     I have a local docker network (dctm-dev) that I use to connect whole Documentum stack and its sub components like postgres etc. 
     This allow me to use docker hostname as FQDN.
     
     ```
     docker network create dctm-dev
     docker run --network dctm-dev --name postgres --hostname postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres:11
     ```
      
  3. Provision Tomcat Docker Container
     Create a tomcat 10 docker container with java 11.
     
     ```
     docker run --network dctm-dev -d --name documentum-otds --hostname documentum-otds -p 9001:8080 tomcat:10
     ```
     
  4. Install OTDS
     a. Download Official 'otds-2210-lnx.tar' file from OpenText Download Centre (under 'All Products')
     b. Extract and fix some shell scripts
        I assume that the directory where tar file has been placed is `/media-files/OTDS`
        ```
        tar -xf otds-2210-lnx.tar 
        sed -i 's/bin\/sh/bin\/bash/g' tools/checkJDBCstring.sh
        ```
     c. Create a response file
        This is a silent installer response file which will be used to setup OTDS.
        ```
        cat << EOF > response.properties
        [Setup]
        Id=OTDS
        Version=22.1.0.3968
        Patch=0
        Basedir=/usr/local/tomcat/temp/
        Configfile=/usr/local/tomcat/temp/setup.xml
        Action=Install
        Log=
        Instance=1
        Feature=All

        [Property]
        INST_GROUP=root
        INST_USER=root
        INSTALL_DIR=/usr/local/OTDS
        TOMCAT_DIR=/usr/local/tomcat
        PRIMARY_FQDN=documentum-otds
        ISREPLICA_TOPOLOGY=0
        OTDS_PASS=password
        IMPORT_DATA=0
        ENCRYPTION_KEY=
        MIGRATION_OPENDJ_URL=
        MIGRATION_OPENDJ_PASSWORD=password
        JDBC_CONNECTION_STRING=jdbc:postgresql://postgres:5432/otds
        JDBC_USERNAME=postgres
        JDBC_PASSWORD=password
        EOF
        ```
     d. Copy files required for next step
     e. Install OTDS
     
  6. Test OTDS Admin Portal


