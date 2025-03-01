# Build Images

    oc new-build https://github.com/openshift-examples/own-apache-container.git \
    --name httpd

    oc new-build https://github.com/openshift-examples/own-apache-container.git \
    --context-dir=anyuid --name httpd-anyuid


# Deploy Images

    oc new-app httpd
    oc expose svc/httpd

    oc new-app httpd-anyuid
    oc expose svc/httpd-anyuid
    # FAIL, because by default it is not allowed to run POD's with uid 0

    # Create service account
    oc create sa anyuid

    # Add privileges to service account to run POD's with uid 0
    oc adm policy add-scc-to-user -z anyuid anyuid 

    oc patch dc/httpd-anyuid --patch '{"spec":{"template":{"spec":{"serviceAccount": "anyuid", "serviceAccountName": "anyuid"}}}}'

# Add persistent volume

```
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    annotations:
        volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/glusterfs
    name: httpd
spec:
    accessModes:
    - ReadWriteMany
    resources:
        requests:
            storage: 1Gi
    storageClassName: glusterfs-ocs
EOF
```

    oc get pvc --watch

    oc set volume dc/httpd --add --name=httpd-volume-1 -t pvc --claim-name=httpd --overwrite  --mount-path=/var/www/html/

    oc set volume dc/httpd-anyuid --add --name=httpd-anyuid-volume-1 -t pvc --claim-name=httpd --overwrite  --mount-path=/var/www/html/

# Rollback 

    oc rollback dc/httpd-anyuid