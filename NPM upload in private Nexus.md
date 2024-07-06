1. create nexus
2. create hosted npm repo
3. add npm bear token realm
4. buld docker and download npm libraries

```
# git clone npm source repo
$ git clone https://github.com/flytux/CineVisionMicroserviceProject.git
$ cd CineVisionMicroserviceProject/frontend

#.yarnrc
cat << EOF >> .yarnrc
yarn-offline-mirror "./npm_packages"
yarn-offline-mirror-pruning true
EOF

#.npmrc
cat << EOF >> .npmrc
registry=http://192.168.45.245:9000/repository/repos/
#echo -n "admin:1" | base64 -d
//192.168.45.245:9000/repository/repos/:_auth="YWRtaW46MQ=="
EOF

$ rm package-lock.json

nerdctl run -it --name builder -v ${PWD}:/app node:18.13.0-alpine sh

$ yarn install

$ ls -al /root/npm_packages

$ cat << EOF >> publish.sh
#!/bin/bash

REPOSITORY=http://192.168.45.245:9000/repository/repos
PACKAGES_PATH=./npm_packages

for package in $PACKAGES_PATH/*.tgz; do
    npm publish --registry=$REPOSITORY $package
done
EOF

$ chmod +x publish.sh

$ ./publish.sh

$ rm -rf node_modules

$ yarn cache clean

$ yarn config set registry http://192.168.45.245:9000/repository/repos/

$ yarn config list

$ yarn install

$ yarn run build

$ yarn start

$ curl -v localhost:3000


```
  
