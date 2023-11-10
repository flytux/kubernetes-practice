### Keycloak SCIM

```bash

# Install Keycloak helm chart

cat << EOF >> values.yaml
auth:
  adminUser: admin
  adminPassword: "admin123"
ingress:
  enabled: true
  ingressClassName: "nginx"
  hostname: keycloak.kw01
postgresql:
  enabled: true
  auth:
    postgresPassword: "postgres"
    username: keycloak
    password: "keycloak"
    database: keycloak
  architecture: standalone
logging:
  output: default
  level: DEBUG
EOF

helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade -i keycloak -f values.yaml keycloak-17.0.0.tgz

# SSL required NONE setting
k exec -it keycloak-postgresql-0 bash
psql -U keycloak
update REALM set ssl_required = 'NONE' where name = 'master';

# Restart Keycloak statefulset
k rollout restart sts keycloak

# Access amdin console
http://keycloak.kw01/admin/master/console/

admin / admin123

```

