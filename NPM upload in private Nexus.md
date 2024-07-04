1. create nexus
2. create hosted npm repo
3. add npm bear token realm
4. buld docker and download npm libraries

```
#.npmrc

registry=http://192.168.45.245:9000/repository/repos/
#echo -n "admin:1" | base64 -d
//192.168.45.245:9000/repository/repos/:_auth="YWRtaW46MQ=="

#.yarnrc

yarn-offline-mirror "./npm_packages"
yarn-offline-mirror-pruning true

yarn install


# rm package-lock.json

# Dockerfile
FROM node:alpine3.20 AS builder
WORKDIR /app
COPY package*.json ./

$ nerdctl build -t npm:builder .

$ nerdctl run -it npm:builder /bin/bash

$ vi .yarnrc
yarn-offline-mirror "./npm_packages"
yarn-offline-mirror-pruning true

$ yarn install

$ ls -al /root/npm_packages

$ vi publish.sh
#!/bin/bash

REPOSITORY=http://%NEXUS_REPO_URL%
PACKAGES_PATH=./npm_packages

for package in $PACKAGES_PATH/*.tgz; do
    npm publish --registry=$REPOSITORY $package
done

$ chmod +x publish.sh

$ ./publish.sh
```
  
