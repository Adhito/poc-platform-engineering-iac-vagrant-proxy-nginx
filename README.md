# Vagrant NGINX Reverse Proxy

This repository provides a Vagrant configuration for setting up an NGINX reverse proxy on an Ubuntu 22.04 virtual machine. The proxy enables SSL termination and forwarding to various backend services running on a private network.

## Features
- Uses Ubuntu 22.04 (Jammy Jellyfish) as the base OS.
- Configures a VirtualBox VM with:
  - 4GB RAM
  - 2 vCPUs
  - Private network IP: `192.168.56.20`
- Installs and configures NGINX as a reverse proxy.
- Generates a self-signed SSL certificate.
- Configures reverse proxy rules for:
  - Kubernetes Dashboard
  - ArgoCD
  - Java Web Application
  - PgAdmin
- Enables automatic redirection from HTTP to HTTPS.

## Prerequisites
Ensure you have the following installed:
- [Vagrant](https://www.vagrantup.com/)
- [VirtualBox](https://www.virtualbox.org/)

## Setup Instructions
1. Clone this repository:
   ```sh
   git clone <repository-url>
   cd <repository-name>
   ```
2. Start the Vagrant VM:
   ```sh
   vagrant up
   ```
3. Access the services through the configured domain names (ensure DNS resolution is properly set up in `/etc/hosts` or your local DNS server):

   | Service                 | URL                                             |
   |-------------------------|-------------------------------------------------|
   | Kubernetes Dashboard    | `https://anindhito.ui.infra.k8sdashboard.local` |
   | ArgoCD                  | `https://anindhito.ui.infra.argocd.local`       |
   | Java Application        | `https://anindhito.ui.infra.javaapp.local`      |
   | PgAdmin                 | `https://anindhito.ui.infra.pgadmin.local`      |

## Modifying the Configuration
- To update the reverse proxy rules, modify the `reverse-proxy` configuration inside `Vagrantfile`.
- Restart NGINX after making changes:
  ```sh
  vagrant ssh
  sudo systemctl reload nginx
  ```

## Destroying the VM
To remove the VM, run:
```sh
vagrant destroy -f
```

## Troubleshooting
- Check NGINX status:
  ```sh
  vagrant ssh
  systemctl status nginx
  ```
- Verify NGINX configuration:
  ```sh
  vagrant ssh
  nginx -t
  ```
- Restart the VM if needed:
  ```sh
  vagrant reload
  ```

## License
This project is licensed under the MIT License.

