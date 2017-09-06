Cloud Security
============

This repository is all about cloud security with [Spring Boot](https://projects.spring.io/spring-boot), 
[Spring Cloud](http://projects.spring.io/spring-cloud) and [Vault](https://www.vaultproject.io).

# Spring Cloud Config Server

## config-server
This project contains the Spring Cloud Config server which must be started like a Spring Boot application before using  
any of the other web applications that require a config server. After starting without a specific profile, it is 
available on port 8888 and will use the configuration files provided in the **config-repo** folder.

Basic auth credentials are user/secret.

## config-client
This Spring Boot based web application exposes the REST endpoints `/`, `/users` and `/credentials`. Based on the active 
Spring profile the configuration files used are not encrypted (**plain**), secured using Spring Config encryption 
functionality (**cipher**), or secured using jasypt (**jasypt**). There is no default profile available so you have to 
provide a specific profile during start. All REST endpoints can be accessed via Swagger at 
**http://localhost:8080/swagger-ui.html**.

### Profile plain
Configuration files are not protected at all, even sensitive configuration properties are available in plain text.

### Profile cipher
This profile uses Config Server functionality to encrypt sensitive properties. It requires either a symmetric or 
asymmetric key. The sample is based on asymmetric encryption and is using a keystore (`server.jks`) which was created by 
executing the following command:

    keytool -genkeypair -alias configserver -keyalg RSA \
      -dname "CN=Config Server,OU=Unit,O=Organization,L=City,S=State,C=Germany" \
      -keypass secret -keystore server.jks -storepass secret
      
The [Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy File](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)
must be installed in order for this to work. The Config Server endpoints help to encrypt and decrypt data:

    curl http://localhost:8888/encrypt -d secretToEncrypt -u user:secret
    curl http://localhost:8888/decrypt -d secretToDecrypt -u user:secret

### Profile jasypt
This profile is using [Jasypt for Spring Boot](https://github.com/ulisesbocchio/jasypt-spring-boot) to secure
sensitive configuration properties. You have to provide an environment variable named `jasypt.encryptor.password` with
the value `config-client-jasypt` to decrypt the database password during application start.

## config-repo
This folder contains all configuration files for all profiles used with the **config-client** and **config-client-vault**
applications.

# Vault

## config-server-vault
This project contains the Spring Cloud Vault server which must be started like a Spring Boot application before using  
the corresponding client web applications **config-client-vault**. [Vault](https://www.vaultproject.io) must be started 
on localhost:

    vault server -config config/vault-local.conf
    export VAULT_ADDR=http://127.0.0.1:8200
    vault init -key-shares=5 -key-threshold=2
    vault unseal [Key 1]
    vault unseal [Key 2]

It must contain the following values:

    vault write secret/config-client-vault name=config-client-vault profile=default

The bootstrap.yml file in the config-server-vault project does use the root token shown during vault init. You have to 
update this token to the one shown during vault initialization.

After starting the Spring Boot application without a specific profile, the server is available on port 8888.

## config-client-vault
This Spring Boot based web application exposes the REST endpoints `/` and `/secrets`. The `/` endpoint provides simple
read access to the values created before. The `/secrets` endpoint provides POST and GET methods to read and write 
individual values to the configured Vault.
    
The bootstrap.yml file in the config-client-vault project does use the root token shown during vault init. You have to 
update this token to the one shown during vault initialization.

## Meta
[![Build Status](https://travis-ci.org/dschadow/CloudSecurity.svg)](https://travis-ci.org/dschadow/CloudSecurity)
[![codecov](https://codecov.io/gh/dschadow/CloudSecurity/branch/develop/graph/badge.svg)](https://codecov.io/gh/dschadow/CloudSecurity)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
