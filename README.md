# Creating and configuring an Gerrit Container authenticating against an LDAP database using docker containers

This project details steps used to set up gerrit using various docker tools.


## Table of Contents
- [Installing LDAP container](#LDAP)
- [Installing LDAP-ADMIN container](#LDAP-ADMIN)

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

####3. create a posix group (e.g. gerrit-users) so as to be able to create user accounts
    a. click ou=groups
    b. click create a child entry
    c. choose Generic: Posix Group
    d. click Create Object button to proceed
    e. on next screen click Commit to persist the changes

####4. create an admin user for use on gerrit
    a. click ou=people
    b. click create a child entry
    c. choose Generic: User Account
    d. fill in gid, lastname, login shell, password
    e. click Create Object button to proceed
    f. on next screen click Commit to persist the changes
####N.B. the first login on gerrit becomes the admin user so choose a cn wisely
