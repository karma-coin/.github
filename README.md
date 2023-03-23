# .github
KarmaCoin project information


## Configuring the Karma Coin Server

### Step 1

Create systemd file for first service /etc/systemd/system/karmacoin-verifier.service 

```
root@kc-ap1-instance:~# cat /etc/systemd/system/karmacoin-verifier.service 
[Unit]
Description=authenticator service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=a
WorkingDirectory=/home/a/karmacoin-verifier/
ExecStart=/home/a/karmacoin-verifier/authenticator.exe

[Install]
WantedBy=multi-user.target
```

Start service
systemctl daemon-reload
systemctl start karmacoin-verifier.service

### Step 2
Create systemd file for first service /etc/systemd/system/karmacoin-server.service 

```
[Unit]
Description=grpc service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=a
WorkingDirectory=/home/a/karmacoin
ExecStart=/home/a/karmacoin/target/debug/server-app -c verifier.yaml

[Install]
WantedBy=multi-user.target
```

Satrt service
systemctl daemon-reload
systemctl start karmacoin-server.service


### Step 3

Install nginx

```
apt install nginx
```

### Step 4

Create configuration for nginx /etc/nginx/conf.d/grpc.conf

```
server {
  server_name api.karmaco.in;
  listen 50055 http2 ;


  location / {
    grpc_pass 127.0.0.1:9080;
  }

}
```

### Step 5

Install certbot 

```
apt install certbot python3-certbot-nginx
```

Generate cert

```
sudo certbot --nginx -d api.karmaco.in
```

Modify /etc/nginx/conf.d/grpc.conf

remove line "listen 50055 http2 ;"
modify line "listen 443 ssl;" to "listen 443 http2 ssl;"

Final view

```
server {
  server_name api.karmaco.in;
  listen 443 http2 ssl;


  location / {
    grpc_pass 127.0.0.1:9080;
  }

  #default_type application/grpc;  

  ssl_certificate /etc/letsencrypt/live/api.karmaco.in/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/api.karmaco.in/privkey.pem;
  include /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

}
```


# Check Status

```
systemctl status certbot.service
```

```
systemctl status nginx.service
```
