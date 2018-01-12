# Creating and configuring an Gerrit Container authenticating against an LDAP database using docker containers

This project details steps used to set up gerrit using various docker tools.


## Table of Contents
1. [Installing an LDAP container](#LDAP)
- [Installing an LDAP-ADMIN container](#LDAP-ADMIN)
- [Configuring the LDAP-ADMIN container](#LDAP-ADMIN CONFIGURATION)
- [Installing a MYSQL container](#MYSQL)
- [Installing a GERRIT container](#GERRIT)

## LDAP
####1.  create the docker container using accenture/adop-ldap
    a. docker run -d --name LDAP -v /servers/ldap/:/var/lib/ldap -p 389:389 -p 636:636 -e SLAPD_DOMAIN=btech.net -e SLAPD_PASSWORD=secret -e SLAPD_FULL_DOMAIN=dc=btech,dc=net -e INITIAL_ADMIN_USER=user -e INITIAL_ADMIN_PASSWORD=secret accenture/adop-ldap:latest

####2. test the created ldap container
    a. docker ps -a  
    if it is up and running then:
    b. docker exec LDAP ldapsearch -x -H ldap://localhost -b ou=people,dc=btech,dc=net -D "cn=admin,dc=btech,dc=net" -w secret
    c. docker exec LDAP ldapsearch -x -H ldap://localhost -b dc=btech,dc=net -D "cn=admin,dc=btech,dc=net" -w secret

## LDAP-ADMIN
####1.  create the docker container using dinkel/phpldapadmin linking it to the LDAP container
    a. docker run -d --name LDAP-ADMIN -p <your-port-number>:80 --link LDAP:ldap -e LDAP_SERVER_HOST=ldap dinkel/phpldapadmin

####2. test the created ldap-admin container
    a. visit <your-host-ip>:<your-port-number> in a browser
    b. login using "cn=admin,dc=btech,dc=net"

## LDAP-ADMIN CONFIGURATION
####1. create a posix group (e.g. gerrit-users) so as to be able to create user accounts
    a. click "ou=groups"
    b. click "create a child entry"
    c. choose "Generic: Posix Group"
    d. click "Create Object" button to proceed
    e. on next screen click "Commit" button to persist the changes

####2. create users for use on gerrit
    a. click "ou=people"
    b. click "create a child entry"
    c. choose "Generic: User Account"
    d. fill in "gid", "lastname", "login shell", "password"
    e. click "Create Object" button to proceed
    f. on next screen click "Commit" to persist the changes
    g. click "Add new attribute" to add "Email" and "displayName" attributes
####N.B. the first login on gerrit becomes the admin user so choose a "cn" wisely

## MYSQL
####1. create the docker container using mysql/mysql-server:5.7
    a. docker run -d --name MYSQL5.7 -p 3306:3306 -v /servers/mysql5.7:/var/lib/mysql mysql/mysql-server:5.7

####2. check its availability
    a. docker ps -a

####3. get its default password
    a. docker logs MYSQL5.7 2>&1 | grep GENERATED

####4. enter container and change default password before you can start using it
    a. docker exec -it MYSQL5.7 mysql -u root -p<GENERATED PASSWORD>
    b. SET PASSWORD for 'root'@'localhost' IDENTIFIED BY 'secret';
    c. create USER 'root'@'%' IDENTIFIED BY 'secret';

####5. create a database for gerrit
    a. create database gerritdb;

## GERRIT

####1. create the docker container using: 
    a. docker run -d --name GERRIT --link MYSQL5.7:db 
	--link LDAP:ldap \
	-p <your-port-number>:8080 -p 29418:29418 \
	-e GITWEB_TYPE=gitiles \
	-e /servers/gerrit:/var/gerrit/review_site \
	-e WEBURL: http:<your-host-ip>:<your-port-number> \
	-e DATABASE_TYPE: mysql \
	-e DATABASE_HOSTNAME:MYSQL \
	-e DATABASE_PORT: 3306 \
	-e DATABASE_DATABASE: gerritdb \
        -e DATABASE_USERNAME: root \
        -e DATABASE_PASSWORD: secret \
        -e AUTH_TYPE: LDAP \
        -e LDAP_SERVER: ldap://ldap:389 \
        -e LDAP_ACCOUNTBASE: ou=people,dc=btech,dc=net \
        -e SMTP_SERVER: <your-smtp-server-address> \
        -e SMTP_SERVER_PORT: <your-smtp-server-port> \
        -e SMTP_ENCRYPTION: <your-smtp-server-encryption-type> \
        -e SMTP_USER: <your-smtp-server-username> \
        -e SMTP_PASS: <your-smtp-server-password> \
        -e SMTP_FROM: <your-gerrit-admin><<your-gerrit-admin-email>> \
        -e USER_NAME: <your-proposed-gerrit-username> \
        -e USER_EMAIL: <your-gerrit-admin><<your-gerrit-admin-email>> \
	openfrontier/gerrit:latest   

####2. check its availability
    a. docker ps -a

####3. test the created gerrit container
    a. visit <your-host-ip>:<your-port-number> in a browser
    b. login using an existing "user account cn" created in ldap

####N.B. the first login becomes a gerrit administrator
