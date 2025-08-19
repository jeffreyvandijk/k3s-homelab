sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl status nginx
sudo apt install libnginx-mod-stream

# /etc/nginx/nginx.conf
load_module /usr/lib/nginx/modules/ngx_stream_module.so;
worker_processes  1;

events {
    worker_connections 1024;
}

stream {
    log_format streamlog '$remote_addr [$time_local] '
                         'SNI="$ssl_preread_server_name" '
                         'protocol=$protocol '
                         'status=$status '
                         'upstream=$upstream_addr '
                         'bytes_sent=$bytes_sent bytes_received=$bytes_received '
                         'session_time=$session_time';

    access_log /var/log/nginx/stream-access.log streamlog;
    error_log  /var/log/nginx/stream-error.log warn;
    map $ssl_preread_server_name $upstream {
        awx.lan            awx_backend;
        registry.lan       registry_backend;
        grafana.lan        grafana_backend;
        default                 reject;
    }

    upstream awx_backend {
        server 192.168.1.20:443;
        server 192.168.1.21:443;
        server 192.168.1.22:443;
    }
    upstream registry_backend {
        server 192.168.1.20:443;
        server 192.168.1.21:443;
        server 192.168.1.22:443;
    }
    upstream grafana_backend {
        server 192.168.1.20:443;
        server 192.168.1.21:443;
        server 192.168.1.22:443;
    }
    upstream k3s_master {
    server 192.168.1.20:6443;
    server 192.168.1.21:6443;
   }

    upstream reject {
        server 127.0.0.1:6666;
    }

    server {
        listen 443;
        proxy_pass $upstream;
        ssl_preread on;
    }
  server {
    listen 6443;
    proxy_pass k3s_master;
  }
}

nginx -t
sudo systemctl restart nginx