### Gradle Airgapped Build

```

# Prepare Gradle Repo

https://github.com/flytux/hwaero-devops/tree/main/gradle-spring

# Set MavenLocal Repo - build.gradle

# Set MavenLocal Plugin Repo - settings.gradle

# Set copy gradle copy to maven local

# Setup Nexus

# Upload dependencies from .m2 to private nexus using mavenimport.sh

# Block outbound traffic to internet

# Libvirt subnet 192.168.122.0/24
$ iptables -I FORWARD -s 192.168.122.0/24 -j DROP
$ iptables -nL
$ iptables -D FORWARD -s 192.168.122.0/24 -j DROP


# Build Gradle in air-gapped enviroment
