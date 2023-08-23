# Setup HAProxy for cluster load balancer of nginx ingress controllers

```bash

# Create haproxy config directory
mkdir -p ~/haproxy/config

# Create haproxy config file
cat << EOF >> $HOME/haproxy/config/haproxy.cfg
defaults
    log global #<-- Edit these three lines
    option tcplog
    mode http

frontend proxynode #<-- Add the following lines
    bind *:80
    stats enable
    stats uri /stats
    default_backend kw01-demo

backend kw01-demo
    balance roundrobin
    server kw01-demo 10.78.15.13:80 check # IP : Port of nginx ingress controller
EOF

# Run docker container
docker run -d --name haproxy -p 80:80 -v /root/haproxy/config:/usr/local/etc/haproxy:ro \
    --sysctl net.ipv4.ip_unprivileged_port_start=0 haproxy:2.3

# Check connection
curl localhost
curl localhost/stats

```

