## Gradle air-gapped build with local maven repository


#### 1) Prepare Gradle Repo

```
https://github.com/flytux/hwaero-devops/tree/main/gradle-spring
```

#### 2) Set MavenLocal repo - build.gradle

```
repositories {

    mavenLocal()
    maven {
        url "http://192.168.122.11:9000/repository/maven-releases"
        allowInsecureProtocol = true
    }
    //mavenCentral(); gradlePluginPortal()
}
```

#### 3) Set MavenLocal plugin repo - settings.gradle

```
pluginManagement {
    repositories {
        maven {
                url "http://192.168.122.11:9000/repository/maven-releases"
                allowInsecureProtocol = true
        }
        //mavenCentral(); gradlePluginPortal()

    }
}
```

#### 4) Set gradle cache copy to maven local

```
// Add gradle cache to mvn repo task
build {
    finalizedBy 'cacheToMavenLocal'
}

task cacheToMavenLocal(type: Copy) {
    from new File(gradle.gradleUserHomeDir, 'caches/modules-2/files-2.1')
    into repositories.mavenLocal().url
    eachFile {
        List<String> parts = it.path.split('/')
        it.path = [parts[0].replace('.','/'), parts[1], parts[2], parts[4]].join('/')
    }
    includeEmptyDirs false
}
```

#### 5) Setup Nexus

```
$ docker run -p 8081:8081 -p 5000:5000 sonatype:nexus3:3.70.1
```

#### 6) Upload dependencies from .m2 to private nexus using mavenimport.sh

```
$ docker run -it -v "$HOME/.m2:/root/.m2" -v "$PWD:/home/gradle/project" -w /home/gradle/project gradel:jdk21-jammy bash
$ gradle build

$ cd ~/.m2/repository
$ ./mavenimport.sh -r http://192.168.122.11:8081/repository/maven-releases

```

#### 7) Block outbound traffic to internet

```
# Libvirt subnet 192.168.122.0/24
$ iptables -I FORWARD -s 192.168.122.0/24 -j DROP
$ iptables -nL
$ iptables -D FORWARD -s 192.168.122.0/24 -j DROP
```

#### 8) Build Gradle in air-gapped enviroment

```
$ gradle jib \
    -Djib.to.image=192.168.122.11:5000/gradle:sample \
    -Djib.to.auth.username=admin \
    -Djib.to.auth.password=1 \
    -Djib.allowInsecureRegistries='true' \
    -DsendCredentialsOverHttp='true' \
    -Djib.from.image=192.168.122.11:5000/eclipse-temurin:21-jre
```

