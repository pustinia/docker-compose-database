# docker-compose-database (oracle 19c, 19.3.0.0 version)
### This is my litte journey for make docker image & compose from oracledaabase github.

I needed oracle docker image for the interface test at work, so I searched on Google as usual.  
However, the Oracle official page was down(This product is no longer maintained on this page,   
and will be removed in the future~~) in dockerhub and the official Oracle database image was nowhere to be found.  
So I write down the process of creating an oracle docker image on there official github.  
It's a story that I thought it would end quickly, but it didn't.  

![image](https://user-images.githubusercontent.com/17061046/218296727-54e91a44-ca41-4542-8fea-deada59d9f60.png)


### 1. Go to the Oracle's official githb, and clone my local desktop.(windows11)
```
git clone https://github.com/oracle/docker-images.git
```
### 2. It is a path to make docker image
```
/docker-images/OracleDatabase/SingleInstance/dockerfiles/
```
### 3. Oh great, There are many versions in there. I need 19.3.0 version
```
11.2.0.2
12.1.0.2
12.2.0.1
18.3.0
18.4.0
19.3.0
21.3.0
buildContainerImage.sh
```
### 4. [RADME](https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/README.md) is great, All the explanations are here. This might end quickly !!!
```
./buildContainerImage.sh -v 19.3.0 -t oracle-19c:19.3.0 -e -i -o '--build-arg SLIMMING=false'
```
### 5. And Run command. (Succes in just one try?)

### 6. Errors !!!! What is it ?
```
 => CACHED [base 1/4] FROM docker.io/library/oraclelinux:7-slim@sha256:68613ad8b1c0cc2746f2ae80ef890b373fa17a8f4ddce4e54  0.0s
 => [internal] load build context                                                                                         0.0s
 => => transferring context: 55.31kB                                                                                      0.0s
 => CACHED [base 2/4] COPY setupLinuxEnv.sh checkSpace.sh /opt/install/                                                   0.0s
 => CACHED [base 3/4] COPY runOracle.sh startDB.sh createDB.sh createObserver.sh dbca.rsp.tmpl setPassword.sh checkDBSta  0.0s
 => CACHED [base 4/4] RUN chmod ug+x /opt/install/*.sh &&     sync &&     /opt/install/checkSpace.sh &&     /opt/install  0.0s
 => ERROR [builder 1/2] COPY --chown=oracle:dba LINUX.X64_193000_db_home.zip db_inst.rsp installDBBinaries.sh /opt/insta  0.0s
------
 > [builder 1/2] COPY --chown=oracle:dba LINUX.X64_193000_db_home.zip db_inst.rsp installDBBinaries.sh /opt/install/:
 ```
 hum.. need more oracle linux file, and go google search. Please give me that file.  
 
### 7. Search that file, and download.
I found [oracledownload](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html) page. and click 'LINUX.X64_193000_db_home.zip' file download.

![image](https://user-images.githubusercontent.com/17061046/218296749-0de8e902-e8d2-4eb2-960f-53b460050012.png)


### 8. Hum.. need to login Oracle. What was my account and password...
Log in with email authentication and download... It's not easy.  
Which folder should I put it in???? It is a question..  

### 9. Ok. I found that folder.
```
~/docker-images/OracleDatabase/SingleInstance/dockerfiles/19.3.0/LINUX.X64_193000_db_home.zip
```

### 10. And run command again.
```
./buildContainerImage.sh -v 19.3.0 -t oracle-19c:19.3.0 -e -i -o '--build-arg SLIMMING=false'
```
- ok. image build complete.

### 11. Let me share my docker-compose.yml
- docker-compose.yml
```
###################################################
# docker run --name <container name> \
# -p <host port>:1521 -p <host port>:5500 -p <host port>:2484 \
# -e ORACLE_SID=<your SID> \
# -e ORACLE_PDB=<your PDB name> \
# -e ORACLE_PWD=<your database passwords> \
# -e INIT_SGA_SIZE=<your database SGA memory in MB> \
# -e INIT_PGA_SIZE=<your database PGA memory in MB> \
# -e INIT_CPU_COUNT=<cpu_count init-parameter> \
# -e INIT_PROCESSES=<processes init-parameter> \
# -e ORACLE_EDITION=<your database edition> \
# -e ORACLE_CHARACTERSET=<your character set> \
# -e ENABLE_ARCHIVELOG=true \
# -e ENABLE_TCPS=true \
# -v [<host mount point>:]/opt/oracle/oradata \
# oracle/database:21.3.0-ee
###################################################
version: '3.5'
services:
  oracle11g:
    
    image: oracle-19c:19.3.0
    container_name: oracle-19c
    environment:
      - ORACLE_SID=ORCLCDB       # SID that should be used (default: ORCLCDB).
      - ORACLE_PWD=admin         # SYS, SYSTEM and PDB_ADMIN password. need to be change
      - ORACLE_PDB=ORCLPDB1      # PDB name that should be used (default: ORCLPDB1).
      - ORACLE_EDITION=enterprise
      - ORACLE_CHARACTERSET=AL32UTF8
      - ORACLE_ALLOW_REMOTE=true
    volumes:
      - oracle19c:/opt/oracle/oradata
    privileged: true
    ports:
      - 1521:1521
      - 5500:5500
      - 2484:2484
volumes:
  oracle19c: {}
```
### 12. docker compose up
- Wait 100% percent for initialization. It takes some time. Do not break this initialization by Ctrl+c key. like me...
```
oracle-19c  | Prepare for db operation
oracle-19c  | 8% complete
oracle-19c  | Copying database files
oracle-19c  | 31% complete
oracle-19c  | Creating and starting Oracle instance
oracle-19c  | 32% complete
oracle-19c  | 36% complete
...
...
oracle-19c  | Completing Database Creation
oracle-19c  | #########################
oracle-19c  | DATABASE IS READY TO USE!
oracle-19c  | #########################
```

### 13. Other few things.
- Make sure your .wslconfig file before you build the oracle docker image. (windows)
- It's recommend 4GB+ memory, 100GB swap size for docker builds.
```
[wsl2]
memory=8GB
processors=2
swap=100GB
localhostForwarding=true
```
- For this memory [issue](https://github.com/oracle/docker-images/issues/458#issuecomment-381359844),  
  It is good to add shared memory setting in docker comopse file.
- The final docker-compose.yml is.....

```
version: '3.5'
services:
  oracle11g:
    shm_size: '2gb'              # oracle default shared memory is 1GB
    image: oracle-19c:19.3.0
    container_name: oracle-19c
    environment:
      - ORACLE_SID=ORCLCDB       # SID that should be used (default: ORCLCDB).
      - ORACLE_PWD=admin         # SYS, SYSTEM and PDB_ADMIN password.
      - ORACLE_PDB=ORCLPDB1      # PDB name that should be used (default: ORCLPDB1).
      - ORACLE_EDITION=enterprise
      - ORACLE_CHARACTERSET=AL32UTF8
      - ORACLE_ALLOW_REMOTE=true
    volumes:
      - oracle19c:/opt/oracle/oradata
    privileged: true
    ports:
      - 1521:1521
      - 5500:5500
      - 2484:2484
volumes:
  oracle19c: {}
```

### 14. finally, Dbeaver setting to login the oracle.
- Dont forget to change the password !!!!  
![image](https://user-images.githubusercontent.com/17061046/218296765-9e993ebb-9f26-4a4a-ac4b-0da9239d2002.png)

