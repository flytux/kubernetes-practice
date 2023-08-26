# Setup HAProxy for cluster load balancer of nginx ingress controllers

```bash

# Create haproxy config directory
mkdir -p ~/haproxy/config

# Create haproxy config file
cat << EOF >> $HOME/haproxy/config/haproxy.cfg
defaults
    log global
    option tcplog

frontend http_front
    mode tcp
    bind *:80
    default_backend http_back

frontend https_front
    mode tcp
    bind *:443
    default_backend https_back

backend http_back
    mode tcp
    server kw01-demo 192.168.121.181:80 check

backend https_back
    mode tcp
    server kw01-demo 192.168.121.181:443 check
EOF

# Run docker container
docker run -d --name haproxy -p 80:80 -v /root/haproxy/config:/usr/local/etc/haproxy:ro \
    --sysctl net.ipv4.ip_unprivileged_port_start=0 haproxy:2.3

# Check connection
curl localhost

```

