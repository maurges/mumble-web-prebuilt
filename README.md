# mumble-web-prebuilt

Pre-built binaries for [mumble-web](https://github.com/Johni0702/mumble-web) and [mumble-web-proxy](https://github.com/Johni0702/mumble-web-proxy)

## License

The original mumble-web repository is distributed under the ISC license, and mumble-web-proxy is distributed under AGPLv3 or later.
For simplicity's sake, this entire repository is licensed under the AGPLv3 license.

## Info

This repository contains a prebuilt version of mumble-web and mumble-web-proxy.

The frontend (mumble-web) was built using Node.js v14.21.3. Its `config.js` file was modified slightly to make usernames defualt to "Anon" + a random number.
Other than the slight modification to `config.js`, it only had its webpack config switched to production mode, and then was built using `npm install`.

The proxy (mumble-web-proxy) was built using rustc 1.66.0 (`stable-x86_64-unknown-linux-gnu`) on Debian 11 running on x86_64 architecture. Therefore, the binary will not work on other architectures.
You need Linux (distro is less relevant) running on x86_64 to use the binary.

Several config snippets are also provided, such as a systemd service file, and an nginx site snippet.

**The any sections past this point assume you're using either Debian or Ubuntu, and have Nginx installed.**

## Installing mumble-web

Copy the contents of the `mumble-web` directory into `/var/www/mumble-web`, and make sure `www-data` has read access to it.

Assuming you're in this repository currently, you can use the following commands to do this:

```sh
# Copy mumble-web into /var/www so that /var/www/mumble-web will contain the frontend
sudo cp -r mumble-web /var/www

# Make www-data the owner of /var/www/mumble-web and give it exclusive read and write access
sudo chown -R www-data /var/www/mumble-web
sudo chmod -R 700 /var/www/mumble-web
```

Now, copy the nginx site config:

```sh
sudo cp mumble-web.conf /etc/nginx/sites-available/mumble-web
```

You must manually edit the nginx config file at `/etc/nginx/sites-available/mumble-web` to set it to your domain/subdomain. Make sure you adjust the LetsEncrypt paths to reflect it, too.

Once you've done that, generate a LetsEncrypt certificate for that domain, if one doesn't already exist. If you don't want to use LetsEncrypt, modify the snippet to do things your way.

You should now be able to enable the site, test the config, and finally restart nginx:

```sh
# Enable the site
sudo ln -s /etc/nginx/sites-available/mumble-web /etc/nginx/sites-enabled

# Test the config
# If this shows you an error, do not restart nginx; go fix your config
sudo nginx -t

# If all is well, restart nginx
sudo systemctl nginx
```

# Installing mumble-web-proxy

First, you need to install some dependencies.

Install `libnice`:

```sh
sudo apt install libnice-dev
```

You also need `libssl`.
Unfortunately, you can't use the version in Ubuntu's repos since mumble-web-proxy needs a specific version to run. In the future, Debian may also have this problem.
For convenience, I've included `libssl` 1.1 in this repository. If you want to download it directly from the Ubuntu archive, you can also get it from `wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb`.

Install `libssl` 1.1:

```sh
# If decide to download it directly from Ubuntu and not use the file in this repository, uncomment the below line:
#wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb

sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb
```

The rest of the guide assumes that you have a user called `mumble-web` that can run the proxy. Here are commands to create that user:

```sh
# Create the user
sudo useradd -m --shell /bin/bash mumble-web

# Restrict access to the user's home directory
sudo chmod 700 /home/mumble-web
```

Before doing anything else, make sure `mumble-web-proxy.toml` is correct for your setup.
By default, it's configured to connect to a local mumble server running on the default mumble port.
If your setup is different, you will need to update this file before proceeding.

Now we need to copy the required files to the user's home directory:

```sh
# Copy the files
sudo cp mumble-web-proxy /home/mumble-web
sudo cp mumble-web-proxy.toml /home/mumble-web

# Make mumble-web the owner of them
sudo chown -R mumble-web /home/mumble-web
```

Take a look at `mumble-web-proxy.service` and ensure it's correct.
It should be if you have followed everything up until this point exactly as written.
If not, modify the file as needed before proceeding.

Next, we need to copy the systemd service file and start it:

```sh
# Copy service file
sudo cp mumble-web-proxy.service /etc/systemd/system

# Enable the service at startup
sudo systemctl enable mumble-web-proxy

# Start the service
sudo systemctl start mumble-web-proxy
```

At this point, mumble-web-proxy should be up and running.
