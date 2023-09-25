# Install
```
helm install sonarqube bitnami/sonarqube -f values.yaml --version 3.3.2 
```

# Grant permission

```
oc adm policy add-scc-to-user privileged -z sonarqube
oc adm policy add-scc-to-user privileged -z sonarqube-postgres
```

# Existing pvc
```
NAME                     STATUS   VOLUME                  CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-minio                Bound    pv-minio                100Gi      RWX                           17d
pvc-sonarqube            Bound    pv-sonarqube            20Gi       RWX                           7h24m
pvc-sonarqube-posegres   Bound    pv-sonarqube-posegres   20Gi       RWX                           7h22m
```



# Create route
```
oc create route edge sonarqube --service sonarqube --hostname 'sonarqube.apps.pump-test.com' --path=/ --insecure-policy=Allow --port=http
```

# Enalbe debug mode
```
diagnosticMode:
  enabled: ture
```

# Run manual after debug mode
```
/opt/bitnami/scripts/sonarqube/entrypoint.sh /opt/bitnami/scripts/sonarqube/run.sh
```


# Error
```
Caused by: java.io.IOException: Cannot create directory '/opt/bitnami/sonarqube/extensions/downloads'.
```

# How to fix
```
initContainers:
 - name: chmod-container
   image: bitnami/os-shell:11-debian-11-r57
   imagePullPolicy: IfNotPresent
   command: ['sh', '-c', 'mkdir -p /opt/bitnami/sonarqube/extensions/downloads && chmod -R 775 /opt/bitnami/sonarqube/extensions']
   securityContext:
     privileged: true
     runAsUser: 0
   volumeMounts:
   - name: sonarqube
     mountPath: /opt/bitnami/sonarqube/extensions
     subPath: extensions
```