###

- kubectl patch configmap/myconfig --type json --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
