![](images/adsmall.png)

# ACE Direct Installation/Configuration Checklist

This checklist describes the installation and configuration procedures for the ACE Direct software. It is the product of the lessons learned from several successful ACE Direct deployments. This document refers to other repos and online sources for some of the low-level installation instructions.

## Prerequisites

> **Warning**: Some prerequisites may take _days_ or even _weeks_ to complete. Factor in this lead time when planning an ACE Direct deployment.

This section describes the important prerequisites to complete _before_ proceeding with an ACE Direct installation.

* This document assumes the installation account to be `/home/acedirect`.  Note that this is the same name as the `/home/acedirect/acedirect` Node servers repo.
* *Acquire domain and subdomain names.* Work with a domain name registrar to register the desired domain and subdomain names. It may take up to *three days* to activate the domain after registration. Domain names _must_ be three-level domain names; this is a requirement by OpenAM. Do not use special characters in the domain names. The suggested domain names for ACE Direct are: _nodeace.domain.com_, _openamace.domain.com_, _stunace.domain.com_, and _sipace.domain.com_. Once domain names are available, proceed with the remaining prerequisites below.
* *Create _A_ records.* Connect your IP addresses to host names with A records ([link](https://www.godaddy.com/help/add-an-a-record-19238)).
* *Update provider peering lists.* Make requests to the phone providers to update their peering lists. It may take up to *two weeks* to fulfill this request.
* *Create website certificates.* Create website certificates for the registered domain name:
  * The certificates should be *wildcard* certs to have flexibility with domain names.
  * Name the certificates _cert.pem_ and _key.pem_.
  * The certificates must have `644` permissions.
  * Place certificates in the expected folders on all the servers, for example: `/home/acedirect/.ssl`
  * If the cert is new, you may have to execute `restorecon` on it, for example: `restorecon -R -v cert.pem`
  * _For local testing only_, you may want to create self-signed certs: `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem`
* *Create the AWS instances for the ACE Direct servers:*
  * Acquire and provision AWS servers for: _nodeace.domain.com_, _openamace.domain.com_, _stunace.domain.com_, and _sipace.domain.com_.
  * On _nodeace_, modify SE Linux: ```sudo setsebool -P httpd_can_network_connect 1``` . You will need to do this after any reboot.
  * Set all servers to the _UTC timezone_.
  * The installation and configuration instructions assume CentOS or RHEL Linux servers. However, other Linux flavors _may_ work with little or no changes to these instructions.
* *Database access.* Secure and test an admin username and password for a MySQL database. This may be either a standard MySQL server or an AWS RDS MySQL service.
* *Internet access.* The installation and configuration require Internet access to download software and other external dependencies on the ACE Direct servers.
* *Install Git.* The _nodeace_ server must have Git. Install it if needed: `yum install git`.

## Installation and configuration

After completing the prerequisites above, continue to ACE Direct installation and configuration. The `autoinstall` repo automates much of the installation and configuration process. 

### Sample Installation

Here is a sample installation example for the `v3.2.0` release on the `nodeace` server:

```
$  # installing ACE Direct...
$  # It is recommended to install Redis, NGINX, MongoDB, and the Node servers on the nodeace server
$  cd /home/acedirect  # your installation folder
$  git clone https://github.com/mitrefccace/autoinstall.git
$  python autoinstall/installer.py -s 'https://github.com/mitrefccace' -b 'v3.2.0' -u '/home/acedirect'
Do you want to install Redis? (y/n): y
...
Do you want to install Nginx? (y/n): y
...
Do you want to install MongoDB? (y/n):y
...
Please select one of the following options for installation. When finished, choose option 0 for configuration:
1. Quick installation & configuration (all servers)
2. Install Agent and Consumer Portals
3. Install ACR-CDR
4. Install Management Portal
5. Install Aserver
6. Install Userver
7. Install Fendesk
8. Install Virtual Agent
9. Disable SE Linux
10. Exit script without configuration

0. Finish installation, begin configuration
 >>  1
...
Virtualagent installation complete.
Disabling SE Linux...
SE Linux has been disabled.
Beginning configuration...
Do you want to go through the configuration process? (y/n):y
Enter the full path to a configuration template file, or press enter to use the
default template:/home/ec2-user/dat/config.json_TEMPLATE
Do you want the configuration file config.json to be base64 encoded? (y/n):n
prompt: common,cleartext:  (true) 
prompt: common,version:  (3.2) 
prompt: common,year:  (2019) 
...
Writing process.json and starting servers of the installed components...
...
$  pm2 update # if needed
$
$  # done, services are up
$  # here's how to manager Node servers with pm2...
$  pm2 stop all  # stop all Node servers
$  pm2 start all  # start all Node servers
$  pm2 status # check status of all Node servers
$  pm2 delete all  # delete Node server processes
$  pm2 start process.json  # add all Node server processes and start them up
```

The following paragraphs describe a manual installation and configuration of ACE Direct.

### _nodeace_

The _nodeace_ server is the main application server for ACE Direct. Complete these instructions on _nodeace_:

* Create a user account named _acedirect_.
* Install and configure NGINX. See the `README.md` file in the `nginx` repo for complete instructions. This repo provides a template for the `nginx.conf` file.
* Install and configure Redis. See the [Redis Quick Start](https://redis.io/topics/quickstart) instructions.
* Install and configure MongoDB. See the [MongoDB installation instructions](https://docs.mongodb.com/manual/administration/install-on-linux/).
* Update the `/etc/hosts` file with the _openamace_ and _nodeace_ information. Here is a sample template:

  ```bash
  127.0.0.1 nodeace.domain.com localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
  <OPENAMACE PRIVATE IP> openamace openamace.domain.com
  <NODEACE PRIVATE IP> nodeace nodeace.domain.com
  ```

* Create the `/home/acedirect/scripts` folder and install the `itrslookup.sh` script. To manually install this script, copy it from the `asterisk` repo:

  ```bash
  $ cd /home/acedirect/scripts
  $ git archive --remote=<Git top-level URL>/asterisk.git HEAD:scripts itrslookup.sh | tar -x
  $ chmod 755 itrslookup.sh
  ```

* Test the `itrslookup.sh` script. For example, to test the 1112223333 VRS number, execute the command below and verify that the IP address and port are shown in the last line of output:

```bash
$ ./itrslookup.sh 1112223333 full
1112223333
SIP URI is 11.22.33.44
SET VARIABLE uri 11.22.33.44:1234
$
```

* Install [Node.js](https://nodejs.org/en/download/package-manager/), [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), wget, [bower](https://bower.io/), [pm2](https://www.npmjs.com/package/pm2), and [apidoc](https://www.npmjs.com/package/apidoc).
* Install and configure the Node.js application servers. Run the automated installer script. Note that the automated installer script also downloads and installs third-party software mentioned above:

```bash
$  cd /home/acedirect  # your installation folder
$  git clone https://github.com/mitrefccace/autoinstall.git
$  python autoinstall/installer.py -s 'https://github.com/mitrefccace' -b 'v3.2.0' -u '/home/acedirect'
```

* When prompted by the `installer.py` script:
  * Select _Option 1_ for a quick installation.
  * Specify the path to the configuration file template: `/home/acedirect/dat/config.json_TEMPLATE`
  * Enter appropriate values for each configuration parameter. See the `parameter_desc.json` file in the `dat` repo for descriptions of each parameter. In many cases, the default value is sufficient.
  * Decide if you want the configuration file to be encoded. If so, set the `cleartext` parameter to `""`. Otherwise, set the `cleartext` parameter to a non-empty string (e.g., `"not encoded"`).

* Verify that the Node.js application servers are running on _nodeace_: ```pm2 status```

### _openamace_

The _openamace_ server is the _identity and access management_ server, implemented with OpenAM. To install and configure _openamace_:

* See the `README.md` file in the `iam` repo for detailed installation and configuration instructions.
* Install Java v1.8. or later. Please be mindful of the new Java [licensing model](https://www.oracle.com/technetwork/java/javase/overview/oracle-jdk-faqs.html). According to Oracle, "Java SE 8 remains free of charge for general purpose desktop and server use" [link](https://www.oracle.com/technetwork/java/javase/overview/faqs-jsp-136696.html). You may optionally install _OpenJDK_ instead. See the `iam` repo for details.

### Database

Some ACE Direct components require database access. The database may be a separate server, or it may be a service like Amazon RDS. Here are full instructions to install and configure the ACE Direct databases:

* See the `dat/README.md` file for additional instructions for installing the database.
* Install a MySQL client on _nodeace_ and _sipace_, for example, `sudo yum install mysql`.
* Install and configure MySQL on a server accessible by _nodeace_ and _sipace_. You may install MySQL as an AWS RDS service instead. Note the admin user and password.
* Modify the `dat/acedirectdefault.sql` script. This script prepares the ACE Direct database:
  * Globally replace `_EXTENSION_PASSWORD_` with the actual extension password from Asterisk.
  * Modify `_ACEDIRECT_PASSWORD_` with a desired password for the _acedirect_ database user.
  * Modify `_ASTERISK_PASSWORD_` with a desired password for the _asterisk_ database user.
* Execute the `dat/acedirectdefault.sql` script from _nodeace_, with the admin user and password. Sample execution assuming the username _admin_ and a sample AWS RDS domain name:

  ```
  $ mysql -u admin -p -h some.aws.rds.amazonaws.com < acedirectdefault.sql
  ```

### Additional installation instructions

* Install and configure the stun server on _stunace_.
* Install and configure Asterisk on _sipace_. See the `asterisk/README.md` file for detailed instructions.
* Install and connect the BusyLight to the agent computer:
  * Connect the USB BusyLight to the agent computer.
  * Download the `busylightapi/exe/lightserver.jar` Java program from the `busylightapi` repo. Run it locally on the Agent's client computer.

# ACE Direct Reboot Checklist

  * Set SE Linux variable: ```sudo setsebool -P httpd_can_network_connect 1```
  * Start NGINX: ```sudo service nginx start```
  * Start Redis: ```sudo service redis start```
  * Start MongoDB: ```sudo service mongod start```
  * Restart Node: ```pm2 restart all```

# ACE Direct Troubleshooting Checklist

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

