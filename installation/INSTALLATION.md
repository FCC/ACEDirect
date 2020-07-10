# ACE Direct Installation

![ACE Direct](../images/adsmall.png)

This document describes how to install, configure, and deploy ACE Direct.

## Prerequisites

> **Note**: Some prerequisites may take _days_ or even _weeks_ to complete. Factor in this lead time when planning an ACE Direct deployment.

Here are important prerequisites to complete _before_ proceeding with an ACE Direct installation:

* *Acquire domain and subdomain names*
  * Contact a domain name registrar to register the desired _domain_ and _subdomain_ names. It may take *several days* to activate the domain after registration.
  * Domain names **must be three-level domain names**. This is a requirement by OpenAM.
  * Do not use special characters in the domain names.
  * The suggested format for domain names are: `acenode.domain.com`, `aceopenam.domain.com`, `acestun.domain.com`, `aceturn.domain.com`, `aceproxy.domain.com`, `acesip.domain.com`, `acekms.domain.com`, and `portal.domain.com`. Note that `portal` is the only public facing server.
* *Create _A_ records* - Connect your IP addresses to host names with _A records_ ([link](https://www.godaddy.com/help/add-an-a-record-19238)).
* *Update provider peering lists* - Contact video/softphone providers to update their peering lists. It could take *two weeks* to fulfill this request.
* *Create website certificates*
  * Certificates should be *wildcard* certs to have flexibility with domain names.
  * Name the certificates `cert.pem` and `key.pem`.
  * Place certificates in the expected folders on all the servers, for example: `/etc/ssl/`. Certificates must have `644` permissions. Remember this location for the configuration step.
  * If the certificate is new, you may need to execute `restorecon`, for example: `restorecon -R -v cert.pem`
  * _For local testing only_, you can create self-signed certs: `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem`
* *Create the AWS instances for the ACE Direct servers*
  * Acquire and provision AWS servers for: `acenode.domain.com`, `aceopenam.domain.com`, `acestun.domain.com`, `aceturn.domain.com`, `aceproxy.domain.com`, `acesip.domain.com`, `acekms.domain.com`, and `portal.domain.com`.
  * On `acenode`, modify SE Linux: `sudo setsebool -P httpd_can_network_connect 1`. You will need to do this after any reboot of the server.
  * Set all servers to the _UTC timezone_. Use `chronyd` or `ntp` to synchronize time across all servers.
  * Servers should be CentOS, RHEL Linux, or Amazon Linux 2. Other Linux flavors _may_ work with little or no changes to these instructions. For this document, CentOS is assumed.
* *Database* - Deploy a MySQL database and note the root user and password. This may be either a standard MySQL server or an AWS RDS MySQL service.
* *Internet access* - An Internet connection is required on some servers to install or update the ACE Direct software. This allows the build processes to download software and other external dependencies.
* *Install Git* - Git is required on to clone the public repos. Install Git, for example on CentOS: `sudo yum install git`.
* *ACE Direct User Account* - Identify a user account on `acenode` for the Node servers, for example `/home/acedirect`. All ACE Direct Node.js servers will install here.

## Installation and configuration

The following sections describe how to install and configure ACE Direct.

### acestun

Install the STUN server on `acestun.domain.com`. See [STUN.md](STUN.md).

### aceturn

Install the TURN server on `aceturn.domain.com`. See [TURN.md](TURN.md).

### strongSwan

Install the `strongSwan` server. See [STRONGSWAN.md](STRONGSWAN.md).

### portal

This is the NGINX server for ACE Direct. This server acts as a _reverse proxy_, hiding internal Node.js servers from public access.

* Log into the `portal.domain.com` server.
* Clone the `nginx` repo.
* Install and configure NGINX. See the the `nginx` repo for complete instructions. This repo provides a template for the `nginx.conf` file.

### aceopenam

The `aceopenam` server is the _identity and access management_ server, implemented with OpenAM. To install and configure `aceopenam`:

* Log into the `aceopenam.domain.com` server.
* Clone the `iam` repo.
* See the `README.md` file in the `iam` repo for detailed installation and configuration instructions.

### acesip

The `acesip.domain.com` server is the Asterisk server. To install Asterisk, clone the `asterisk` repo and follow the installation instructions.

### aceproxy

The `aceproxy.domain.com` server is the SIP proxy server. Clone the `kamailio` repo and follow the installation instructions.

### acekms

The `acekms.domain.com` server is the Kurento media server. Clone the `kurento` repo and follow the installation instructions.

Additionally, install the videomail server on `acekms`. Clone the `kurento-asterisk-servlet` and follow the instructions.

### acenode

The **acenode** server hosts several Node.js and application servers. Here are instructions to deploy **acenode**:

1. Install Node.js `v10.16.0`, npm `6.9.0`, and pm2 `4.4.0`.
1. Install `git`.
1. Create an ACE Direct user account to host the Node.js servers, for example `acedirect`.
1. Install and configure Redis. See [Redis Quick Start](https://redis.io/topics/quickstart).
1. Install and configure MongoDB. See [MongoDB installation instructions](https://docs.mongodb.com/manual/).
1. Update the `/etc/hosts` file with the private IP addresses of `acenode.domain.com`, `aceopenam.domain.com`, `acestun.domain.com`, `aceturn.domain.com`, `aceproxy.domain.com`, `acesip.domain.com`, `acekms.domain.com`, and `portal.domain.com`.
1. From the terminal, go into the ACE Direct user account: `cd /home/acedirect`).
1. Clone the following repos:

    * `acedirect`
    * `acedirect-kurento`
    * `acr-cdr`
    * `aserver`
    * `dat`
    * `fendesk`
    * `managementportal`
    * `userver`

1. Configure ACE Direct

  ```bash
  $  cd dat
  $
  $  cp config.json_TEMPLATE config.json  # first time only!
  $  vi config.json  # configure (see dat/parameter_desc.json for help)  
  ```

1. Build repos - Go into each of these folders, and execute `npm run build`:

    * `acedirect`
    * `acedirect-kurento` (if this fails, try `npm run build2`)
    * `acr-cdr`
    * `aserver`
    * `fendesk`
    * `managementportal`
    * `userver`

1. `acedirect-kurento` configuration

    * Edit/configure the following files:

      ```bash
      $  cd acedirect-kurento
      $
      $  vi confs/kurento/WebRtcEndpoint.conf.ini
      $  vi src/config/development.json
      $  vi src/config/db.js
      ```

    * Configure DB:

      ```bash
      $  cd acedirect-kurento
      $
      $  npm run sequelize db:migrate
      ```

1. Starting/stopping ACE Direct Node.js on `acenode`:

  ```bash
  $  cd /home/acedirect  # ACE Direct user account
  $
  $  pm2 start process.json  # first time
  $  pm2 restart all  # subsequent
  $  pm2 stop all  # stop
  $  pm2 stop 0  # stop ID 0
  $  pm2 delete all  # remove all servers from pm2
  ```

### Database Server

Here are full instructions to install and configure the ACE Direct databases:

1. Install MySQL Server Version `5.6.37` or a compatible version.
1. Install a MySQL client on `acenode`, for example, `sudo yum install mysql`.
1. Modify the `dat/acedirectdefault.sql` script. This script prepares the ACE Direct database:

    * Globally replace `_EXTENSION_PASSWORD_` with the actual extension password from Asterisk.
    * Modify `_ACEDIRECT_PASSWORD_` with a desired password for the `acedirect` database user.
    * Modify `_ASTERISK_PASSWORD_` with a desired password for the `asterisk` database user.
    * Modify `_MEDIASERVER_PASSWORD_` with a desired password for the `media_server` database user.

1. Execute the `dat/acedirectdefault.sql` script from `acenode`, with the MySQL admin user and password. Sample execution assuming the username `admin` and a sample AWS RDS domain name:

  ```bash
  $  sudo yum install mysql  # install a MySQL client if it's not there
  $
  $  mysql -u admin -p -h some.aws.rds.amazonaws.com < acedirectdefault.sql
  ```

### Additional installation instructions

* Install and configure the stun server on _stunace_.
* Install and configure Asterisk on _sipace_. See the `asterisk/README.md` file for detailed instructions.
* Install and connect the BusyLight to the agent computer:
  * Connect the USB BusyLight to the agent computer.
  * Download the `busylightapi/exe/lightserver.jar` Java program from the `busylightapi` repo. Run it locally on the Agent's client computer.

## ACE Direct Reboot Checklist

After rebooting servers, ACE Direct requires starting services in a specific order. Some services start automatically on reboot. Here is this required order:

* `aceturn` - TURN
* `acestun` - STUN
* `acesip` - Asterisk
* `aceproxy` - SIP proxy
* `aceopenam` - OpenAM
* `portal` - NGINX
* `acekms` - media server and `kurento-asterisk-servlet`
* `acenode` - Node.js
  * Set SE Linux variable: `sudo setsebool -P httpd_can_network_connect 1`
  * Start Redis: `sudo service redis start`
  * Start MongoDB
  * Start all Node.js servers: `pm2 start process.json` or `pm2 start all`

## ACE Direct Troubleshooting Checklist

* Manage Node services with `pm2`:

  * Run `pm2 status` to check the status of all Node servers:

    * Are all `status` fields `online` (OK)? If not, errors are present.
    * Are any `restart` counts periodically increasing? If so, errors are present.

  * Stop all Node services: `pm2 stop all`
  * Start all Node services: `pm2 start all`
  * Restart all Node services: `pm2 restart all`
  * Delete all Node services from `pm2` management: `pm2 delete all`
  * Add and start all Node services to `pm2`: `pm2 start process.json`

* Set the `common:debug_level` parameter in `/home/acedirect/dat/config.json` to *ALL* to receive all messages in the log files.
* Check the `logs` folder in each application folder for errors or warnings: `ls /home/acedirect/*/logs/*.log`
* Verify that OpenAM, Redis, MongoDB, NGINX, and MySQL are running.
* Does the BusyLight device respond? Try the self-test mode on the `lightserver.jar` UI.
* Verify that the `/etc/hosts` file is configured correctly.
* Verify that the `/etc/nginx/nginx.conf` file is configured correctly.
* Verify that `/home/acedirect/dat/config.json` is configured correctly.
* Check if `asterisk` is publicly accessible: Visit `https://ASTERISK_FQDN/ws`. The page should display `Upgrade Required`.
* Management Portal installation - for any `lodash` errors, try installing the `lodash` library globally as root: `sudo npm install lodash -g`.
* NGINX cannot proxy to the NODE server - when using FQDNs for ACEDirect in `/etc/nginx/nginx.conf`, the FQDNs may force traffic through a proxy. To resolve this, map the FQDN to the private IP instead using a private host zone. *Or*, simply use private IP addresses in place of FQDN in `/etc/nginx/nginx.conf` for the ACEDirect, ManagementPortal, and ace (OpenAM) paths.
* Install MongoDB on RHEL - if the `installer.py` script fails to install MongoDB on RHEL, try `sudo yum install -y mongodb-org` .
* No CDR records in the Management Portal - Make sure Asterisk is configured to have the MySQL database credentials, CDR database name, and CDR table name. Also make sure that the ODBC C library is installed on the Asterisk server; this library is normally installed by the automated installation script.
* Consumer portal cannot reach Asterisk; ERR_CONNECTION_REFUSED - make sure Asterisk is configured to use valid certificates.
* Cannot connect to portals - possibly remap the elastic IPs or try running `nslookup` on the NGINX FQDN and verify its FQDN and public IP.
* NGINX errors when trying to connect to portals, but all servers are up and running - make sure all servers have the correct time, synced with each other.
