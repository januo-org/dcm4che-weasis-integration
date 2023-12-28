# dcm4che-weasis-integration

_manual pacs connector deployment_

```yaml
---
version: "3"
services:
  ldap:
    image: dcm4che/slapd-dcm4chee:2.6.3-30.0
    container_name: ldap
    ports:
      - "389:389"
    volumes:
      - /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data
      - /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M
  db:
    image: dcm4che/postgres-dcm4chee:15.2-30
    container_name: db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
    depends_on:
      - ldap
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/db:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M
  arc:
    image: dcm4che/dcm4chee-arc-psql:5.30.0
    container_name: arc
    ports:
      - "8080:8080"
      - "8443:8443"
      - "9990:9990"
      - "9993:9993"
      - "11112:11112"
      - "2762:2762"
      - "2575:2575"
      - "12575:12575"
    environment:
      WILDFLY_ADMIN_USER: admin
      WILDFLY_ADMIN_PASSWORD: admin
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
      WILDFLY_DEPLOY_UI: "true"
      WILDFLY_WAIT_FOR: ldap:389 db:5432
      WILDFLY_EXECUTER_MAX_THREADS: 200
      WILDFLY_CHOWN: /storage
      # JAVA_OPTS: -Xms512m -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M  -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Duser.timezone=UTC -Djava.net.preferIPv4Stack=true -Djava.awt.headless=true -Duser.language=en -Duser.region=US -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -Dsun.net.client.defaultConnectTimeout=10000 -Dsun.net.client.defaultReadTimeout=30000 -Djava.util.concurrent.ForkJoinPool.common.parallelism=2 -Djava.security.egd=file:/dev/urandom -Djava.rmi.server.hostname=192.168.1.6 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.rmi.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.host=0.0.0.0
      JAVA_OPTS: -Xms512m -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M -Djboss.tx.node.id=arc -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
    depends_on:
      - ldap
      - db
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone
      - /var/local/dcm4chee-arc/storage:/storage
```


Automatic deployment with weasis connector


```Dockerfile
FROM dcm4che/dcm4chee-arc-psql:5.30.0
COPY weasis-pacs-connector.war /docker-entrypoint.d/deployments
COPY weasis.war /docker-entrypoint.d/weasis.war
```

_Build command_

```bash
docker build -t jjino/dcm4chee-arc-psql-with-weasis-pacs-connector:5.30.0
docker push jjino/dcm4chee-arc-psql-with-weasis-pacs-connector:5.30.0
```


_deployment_

```yml
---
version: "3"
services:
  ldap:
    image: dcm4che/slapd-dcm4chee:2.6.3-30.0
    container_name: ldap
    ports:
      - "389:389"
    volumes:
      - /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data
      - /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M
  db:
    image: dcm4che/postgres-dcm4chee:15.2-30
    container_name: db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
    depends_on:
      - ldap
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/db:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M
  arc:
    image: docker.io/library/dcm4chee-arc-psql-with-weasis-pacs-connector:5.30.0
    container_name: arc
    ports:
      - "8080:8080"
      - "8443:8443"
      - "9990:9990"
      - "9993:9993"
      - "11112:11112"
      - "2762:2762"
      - "2575:2575"
      - "12575:12575"
    environment:
      WILDFLY_ADMIN_USER: admin
      WILDFLY_ADMIN_PASSWORD: admin
      POSTGRES_DB: pacsdb
      POSTGRES_USER: pacs
      POSTGRES_PASSWORD: pacs
      WILDFLY_DEPLOY_UI: "true"
      WILDFLY_WAIT_FOR: ldap:389 db:5432
      WILDFLY_EXECUTER_MAX_THREADS: 200
      WILDFLY_CHOWN: /storage
      JAVA_OPTS: -Xms512m -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M -Djboss.tx.node.id=arc -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
    depends_on:
      - ldap
      - db
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone
      - /var/local/dcm4chee-arc/storage:/storage
````

_Record creation_

```bash
docker run --rm --network=host dcm4che/dcm4che-tools:5.30.0 storescu -cDCM4CHEE@192.168.1.6:11112 /opt/dcm4che/etc/testdata/dicom
```

_console URLs_

```bash
http://192.168.1.6:8080/dcm4chee-arc/ui2
http://192.168.1.6:9990
http://192.168.1.6:8080/weasis-pacs-connector/weasis?patientID=583295

```

_Endpoint URLs_

```console
1. DICOM QIDO-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
2. DICOM STOW-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
3. DICOM WADO-RS Base URL: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/rs
4. DICOM WADO-URI: http://192.168.1.6:8080/dcm4chee-arc/aets/DCM4CHEE/wado
5. IHE XDS-I Retrieve Imaging Document Set: http://192.168.1.6:8080/dcm4chee-arc/xdsi/ImagingDocumentSource
```

## Documentations

1. [weasis docs](https://github.com/nroduit/weasis-pacs-connector#launch-weasis-with-other-parameters)
2. [dcm4che docker preparation docs](https://github.com/dcm4che-dockerfiles/dcm4chee-arc-psql#deploy-additional-applications)
3. [dcm4che-arc-light docs](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Get-Started-Tutorials)

