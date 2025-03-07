Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"  # Ubuntu 22.04 LTS
  config.vm.hostname = "devproxynginx01"
  config.vm.network "private_network", ip: "192.168.56.20"

  ## Provisioning: Set CPU, RAM and Name
  config.vm.provider "virtualbox" do |vb|
    vb.name = "DEVPROXYNGINX01"
    vb.memory = "4096"  # 4GB RAM
    vb.cpus = 2         # 2 vCPUs
    vb.customize ["modifyvm", :id, "--groups", "/Development Proxy NGINX Vagrant"]
  end

  # Provisioning: Install NGINX
  config.vm.provision "shell", inline: <<-SHELL
    apt update && apt install -y nginx
  SHELL

  # Provisioning: Generate Self-Signed SSL Certificate
  config.vm.provision "shell", inline: <<-SHELL
    sudo mkdir -p /etc/nginx/ssl
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout /etc/nginx/ssl/nginx-selfsigned.key \
        -out /etc/nginx/ssl/nginx-selfsigned.crt \
        -subj "/C=US/ST=State/L=City/O=Company/OU=Org/CN=anindhito.ui.infra.local"
  SHELL

  # Provisioning: Configure Reverse Proxy
  config.vm.provision "shell", inline: <<-SHELL
    cat <<EOF > /etc/nginx/sites-available/reverse-proxy
    server {
        listen 443 ssl;
        server_name anindhito.ui.infra.k8sdashboard.local;
        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
        location / {
            proxy_pass https://192.168.56.10:30001;
        }
    }

    server {
        listen 443 ssl;
        server_name anindhito.ui.infra.argocd.local;
        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
        location / {
            proxy_pass https://192.168.56.10:30002;
        }
    }

    server {
        listen 443 ssl;
        server_name anindhito.ui.infra.javaapp.local;
        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
        location / {
            proxy_pass http://192.168.56.10:8080;
        }
    }

    server {
        listen 443 ssl;
        server_name anindhito.ui.infra.pgadmin.local;
        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
        location / {
            proxy_pass http://192.168.56.10:30002;
        }
    }

    server {
        listen 80;
        server_name anindhito.ui.infra.k8sdashboard.local anindhito.ui.infra.argocd.local anindhito.ui.infra.javaapp.local anindhito.ui.infra.pgadmin.local;
        return 301 https://$host$request_uri;
    }
EOF
  SHELL

  # Provisioning: Enable NGINX reverse proxy and reload
  config.vm.provision "shell", inline: <<-SHELL
    if [ ! -L /etc/nginx/sites-enabled/reverse-proxy ]; then
        ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
    fi

    if [ -f /etc/nginx/sites-enabled/default ]; then
        rm /etc/nginx/sites-enabled/default
    fi

    systemctl start nginx
    nginx -t && systemctl reload nginx
  SHELL
end
