### Keycloak SCIM

```bash

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: keycloak
spec:
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-keycloak-keycloak
  template:
    metadata:
      creationTimestamp: null
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-keycloak-keycloak
      namespace: keycloak
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - "curl https://lab.libreho.st/libre.sh/scim/keycloak-scim/-/jobs/artifacts/main/raw/build/libs/keycloak-scim-1.0-SNAPSHOT-all.jar?job=package
          -Lo /opt/keycloak/providers/keycloak-scim-1.0-SNAPSHOT-all.jar; \n /opt/keycloak/bin/kc.sh
          start-dev"
        env:
        - name: KC_DB
          value: postgres
        - name: KC_DB_URL_HOST
          value: postgres
        - name: KC_DB_USERNAME
          value: keycloak
        - name: KC_DB_PASSWORD
          value: keycloak
        - name: KEYCLOAK_ADMIN
          value: admin
        - name: KEYCLOAK_ADMIN_PASSWORD
          value: admin123
        image: quay.io/keycloak/keycloak:18.0.0
        imagePullPolicy: Always
        name: keycloak
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          privileged: false
          readOnlyRootFilesystem: false
          runAsNonRoot: false
        stdin: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        tty: true
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: pv-keycloak-providers
        persistentVolumeClaim:
          claimName: pvc-keycloak-providers
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: keycloak
spec:
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-keycloak-postgres
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-keycloak-postgres
      namespace: keycloak
    spec:
      containers:
      - env:
        - name: POSTGRES_USER
          value: keycloak
        - name: POSTGRES_PASSWORD
          value: keycloak
        image: postgres:16
        imagePullPolicy: Always
        name: postgres
        ports:
        - containerPort: 5432
          name: postgres
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          privileged: false
          readOnlyRootFilesystem: false
          runAsNonRoot: false
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: pv-postgres
      volumes:
      - name: pv-postgres
        persistentVolumeClaim:
          claimName: pvc-postgres
---
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  namespace: keycloak
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    workload.user.cattle.io/workloadselector: apps.deployment-keycloak-keycloak
  type: ClusterIP
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: keycloak
spec:
  ports:
  - name: postgres
    port: 5432
    protocol: TCP
    targetPort: 5432
  selector:
    workload.user.cattle.io/workloadselector: apps.deployment-keycloak-postgres
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak
  namespace: keycloak
spec:
  ingressClassName: nginx
  rules:
  - host: keycloak.kw01
    http:
      paths:
      - backend:
          service:
            name: keycloak
            port:
              number: 8080
        path: /
        pathType: Prefix
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-postgres
  namespace: keycloak
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-path

---
./kcadm.sh config credentials --server http://localhost:8080 --realm master --user admin

./kcadm.sh update realms/master -s sslRequired=NONE


Dockerfile

FROM quay.io/keycloak/keycloak:18.0.0
RUN  curl https://lab.libreho.st/libre.sh/scim/keycloak-scim/-/jobs/artifacts/main/raw/build/libs/keycloak-scim-1.0-SNAPSHOT-all.jar?job=package -Lo /opt/keycloak/providers/keycloak-scim-1.0-SNAPSHOT-all.jar


https://lab.libreho.st/libre.sh/scim/keycloak-scim

```
---

```bash
version: "3"

services:
    rocketchat:
        image: registry.rocket.chat/rocketchat/rocket.chat:4.8.1
        command: >
            bash -c
              "for i in `seq 1 30`; do
                node main.js &&
                s=$$? && break || s=$$?;
                echo \"Tried $$i times. Waiting 5 secs...\";
                sleep 5;
              done; (exit $$s)"
        restart: unless-stopped
        volumes:
            - uploads:/app/uploads
        environment:
            - PORT=3000
            - ROOT_URL=http://localhost:3000
            - MONGO_URL=mongodb://mongo:27017/rocketchat
            - MONGO_OPLOG_URL=mongodb://mongo:27017/local
            # - ADMIN_USERNAME=test
            # - ADMIN_PASS=test
            # - ADMIN_EMAIL=test@example.com
        depends_on:
            - mongo
        ports:
            - 3000:3000

    mongo:
        image: mongo:4.4
        restart: unless-stopped
        volumes:
            - db:/data/db
        command: mongod --oplogSize 128 --replSet rs0
        ports:
            - 27017:27017
    mongo-init-replica:
        image: mongo:4.4
        command: >
            bash -c
              "for i in `seq 1 30`; do
                mongo mongo/rocketchat --eval \"
                  rs.initiate({
                    _id: 'rs0',
                    members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
                s=$$? && break || s=$$?;
                echo \"Tried $$i times. Waiting 5 secs...\";
                sleep 5;
              done; (exit $$s)"
        depends_on:
            - mongo

volumes:
    uploads:
    db:
```
