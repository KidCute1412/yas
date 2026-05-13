# Install and configure Nginx reverse proxy

Step 1: Install Nginx on the VM

Run on the VM:

```
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

Check:

```
sudo systemctl status nginx
```

Step 2: Get Minikube IP

```
minikube ip
```

Example output:

```
192.168.49.2
```

Use this IP in the next step.

Step 3: Create the Nginx reverse proxy

Run:

```
sudo vi /etc/nginx/sites-available/yas
```

Paste this content:

```
server {
    listen 80;
    server_name
        identity.yas.local.com
        backoffice.yas.local.com
        storefront.yas.local.com
        api.yas.local.com
        kibana.yas.local.com
        pgadmin.yas.local.com
        akhq.yas.local.com
        grafana.yas.local.com;

    location / {
        proxy_pass http://192.168.49.2:80;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Replace 192.168.49.2 with the real output of:

```
minikube ip
```

Step 4: Enable the config

```
sudo ln -sf /etc/nginx/sites-available/yas /etc/nginx/sites-enabled/yas
sudo nginx -t
sudo systemctl reload nginx
```

If `nginx -t` is OK, continue.

Step 5: Open Google Cloud firewall port 80

In the Google Cloud VM firewall, open:

TCP 80

Source: restrict to your IP if possible.

If you are only testing, you can open 0.0.0.0/0 but it is less secure.

Step 6: Update hosts file on Windows

File:

C:\Windows\System32\drivers\etc\hosts

Add:

```
104.199.151.243 identity.yas.local.com
104.199.151.243 backoffice.yas.local.com
104.199.151.243 storefront.yas.local.com
104.199.151.243 api.yas.local.com
104.199.151.243 kibana.yas.local.com
104.199.151.243 pgadmin.yas.local.com
104.199.151.243 akhq.yas.local.com
104.199.151.243 grafana.yas.local.com
```

Then on Windows run:

```
ipconfig /flushdns
```
