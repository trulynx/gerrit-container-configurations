# Creating and configuring an Gerrit Container authenticating against an LDAP database using docker containers

This project details steps used to set up gerrit using various docker tools.


## Table of Contents
1. [Installing an LDAP container](#LDAP)
- [Installing an LDAP-ADMIN container](#LDAP-ADMIN)
- [Configuring the LDAP-ADMIN container](#LDAP-ADMIN CONFIGURATION)
- [Installing a MYSQL container](#MYSQL)

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
    a. docker run -d --name LDAP-ADMIN -p 28086:80 --link LDAP:ldap -e LDAP_SERVER_HOST=ldap dinkel/phpldapadmin

####2. test the created ldap-admin container
    a. visit <your-host-ip>:28086 in a browser
    b. login using cn=admin,dc=btech,dc=net

## LDAP-ADMIN CONFIGURATION
####1. create a posix group (e.g. gerrit-users) so as to be able to create user accounts
    a. click ou=groups
    b. click create a child entry
    c. choose Generic: Posix Group
    d. click Create Object button to proceed
    e. on next screen click Commit to persist the changes

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
    b. ALTER USER 'root'@'localhost' IDENTIFIED BY 'secret';
    c. create USER 'root'@'%' IDENTIFIED BY 'secret';


