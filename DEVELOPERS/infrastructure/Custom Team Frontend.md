# Set up custom team site

## Create the server

1. Log in to the [Google Cloud Platfrom](https://console.cloud.google.com) console and go to Compute Engine.

2. Click the "Create Instance" button.

3. Choose the following configuration options:

    - **Set the name**: Under **Name**, insert `[TEAM_NAME]-frontend` (all lower case). This is merely a convention, and has no particular programmatic effect.

    - **Set the region**: Under **Region**, choose `us-central1`.  Zone is somewhat flexible, although `us-central1-c` is preferred.

    - **Set the machine type**: Under **Machine Type** choose  `1 vCPU` and `3.75 GB memory`.

    - **Choose the OS**: Under **Boot Disk** click the `Change` button.  For the OS choose `Ubuntu 18.04 LTS` (WARNING: do NOT choose `Ubuntu 18.04 LTS Minimal`, or you'll have to install a LOT more software).  For the **Boot disk type** choose the `Standard persistent disk` and set the size to `10 GB` (likely the default).

    - **Allow Traffic**: Under **Firewall** click `Allow HTTP traffic` and `Allow HTTPS traffic`, so that they are both selected.

    - **Create an External IP**: Click the `Management, security, disks, networking, sole tenancy` text to open the submenu.  The click on `Networking` to switch to the networking tag.  Click the pencil under **network interfaces** to open the Network interface modal.  Click the **External IP** dropdown and change from `Ephemeral` to `Create IP Address`.  This will open up a `Reserve a new static IP address` modal.  Enter the name of `[TEAM_NAME]-frontend` (again, this a convention rather than a strict requirement).  Make sure under **Network Service Tier** that `Premium` is selected.  Then click the `Done` button.  Copy and save the new External IP Address.

4. Click `Create`.

## Configure DNS

1. Log in to the [AWS Console](https://console.aws.amazon.com) and click on Route 53.  Choose the `adv.gg` Hosted Zone.

2. Click `Create Record Set`.

3. Choose the following configuration options.

    - **Name**: enter the desired subdomain in all lower case.  This is likely team name or some variation thereof.

    - **Type**: select `A - IPv4 address`.

    - **Value**: Insert the IP address you created in the previous section.

4. Click `Create`.

## Configure the server

1. Make sure you have the `advocate_rsa` key in your `~/.ssh` folder.  If you don't have it, get it from another Advocate developer.

2. SSH into the server using

```bash
ssh YOUR_GCP_USERNAME@EXTERNAL_IP
```

3. Create the `advocate` user

    - Enter `sudo adduser advocate`

    - Enter `advocate` as the password

    - Skip the Name/room number stuff, and select `y`

4. Add passwordless sudo to the `advocate` user (NOTE: this step is not strictly necessary, but will make the rest of the server configuration at lot easier):

    - enter `sudo visudo`

    - under the `# User privilege specification` section, add this line:

        ```
        advocate   ALL=(ALL) NOPASSWD:ALL
        ```

5. Switch to the advocate user: `su advocate`

6. Copy the `.ssh` folder to the `advocate` home directory and give it proper permissions.  This will make it easier to reconnect to the server later:

    ```bash
    sudo cp -r /home/YOUR_GCP_USERNAME/.ssh /home/advocate/
    sudo chown -R advocate:advocate /home/advocate/.ssh
    ```

7. Copy the `adv_frontend_rsa` contents into `/home/advocate/.ssh/adv_frontend_rsa`; If you don't have it, get it from another Advocate developer. Set it to the proper permissions with:

    ```bash
    sudo chmod 600 /home/advocate/.ssh/adv_frontend_rsa
    ```

8. Change the permissions on the `/srv/` directory to be owned by `advocate`: 

    ```bash
    sudo chown advocate:advocate /srv
    ```

9. Clone the github repo:

    ```bash
    eval `ssh-agent -s`
    ssh-add ~/.ssh/adv_frontend_rsa
    git clone -b master git@github.com:AdvocatesInc/advocate-frontend /srv/adv
    ```
    
    Make sure the theme with [TEAMNAME] exists on the `master` branch of the `advocate-frontend` repo. If it does not, you'll see an error in step 12

10. Install node, npm, yarn, and pm2

    ```bash
    sudo apt-get update
    sudo apt-get install yarn
    sudo apt-get install nodejs npm

    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    sudo npm install pm2@latest -g
    ```

11. Add necessary environment variables to the end of `~/.profile`

    ```bash
    echo 'export ADVOCATE_THEME_NAME=[THEME_NAME]' >> ~/.profile
    echo 'export LOGIN_CALLBACK_URL=https://callbacks.adv.gg/' >> ~/.profile
    echo 'export ADVOCATE_API_URL=https://api.adv.gg/v1/' >> ~/.profile
    echo 'export ADVOCATE_TWITCH_CLIENT_ID=8r7kzzgfvq7fkkfonfwnuixpscv48c' >> ~/.profile
    source ~/.profile
    ```

    except, of course, replace `[THEME_NAME]` with the name of the theme, as it exists in the `advocate-frontend` repo.

12. Install dependencies, build the app

    ```bash
    cd /srv/adv/
    yarn install
    npm run build
    ```

13. Install and configure nginx

    ```bash
    sudo apt-get install nginx
    mkdir /srv/server_configs
    tee /srv/server_configs/express_nginx.conf > /dev/null << EOL
    server {
        server_name TEAM_NAME.adv.gg;

        charset     utf-8;

        root /var/www/html;

        # max upload size
        client_max_body_size 75M;

        # redirect links in the form "https://TEAMNAME.adv.gg/a/LINKCODE" to go.adv.gg
        rewrite ^(/a/)(.*)$  https://go.adv.gg/$2 permanent;

        location / {
            proxy_pass http://localhost:8080;
            proxy_http_version 1.1;
            proxy_set_header Upgrade \$http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host \$host;
            proxy_cache_bypass \$http_upgrade;
        }
    }
    EOL
    sudo ln -s /srv/server_configs/express_nginx.conf /etc/nginx/sites-enabled/
    sudo systemctl restart nginx
    ```

    Of course, replacing `TEAM_NAME` with subdomain you created in a previous step

14. Start pm2

    ```bash
    cd /srv/adv
    pm2 start dist/server.js
    sudo env PATH=$PATH:/usr/bin /usr/local/lib/node_modules/pm2/bin/pm2 startup systemd -u advocate --hp /home/advocate
    ```

15. Configure HTTPS

    ```bash
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt install python-certbot-nginx
    sudo certbot --nginx -d TEAM_NAME.adv.gg
    ```

    when the option comes up for `No redirect`/`Redirect`, choose the `Redirect` option (2).
