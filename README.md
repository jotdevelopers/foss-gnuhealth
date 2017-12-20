# GNU Health Installation

The installation was tested using the following configuration:

- OS: Debian GNU/Linux amd64
- RAM: 1 GB
- Hard drive: 20 GB
- NIC: 1x Gigabit Ethernet

## Install dependencies

The following configuration needs to be applied before installing GNU Health.

```
# apt install build-essential python-dev python-pip \
    libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev python-ldap bsdtar \
    python-imaging postgresql postgresql-server-dev-all libjpeg-dev
```

## Create a dedicated user

This user will be used by the application:

```
# adduser gnuhealth
```

## Configuration of PostgreSQL

### Change PostgreSQL's authentication method

Modify the file `/etc/postgresql/9.6/main/pg_hba.conf` according to the following:

```
# Database administrative login by Unix domain socket
local   all             postgres                                trust

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
```

### Create user for the database

The following command changes the PostgreSQL administrative user and gives permission to the user *gnuhealth* created before:

```
# su - postgres -c "createuser --createdb --no-createrole --no-superuser gnuhealth"
```

Restart PostgreSQL service after the modification:

```
# systemctl restart postgresql
```

## Install GNU Health

Continue the installation with the *gnuhealth* user:

```
# su - gnuhealth
```

Download GNU Health installation file from the [official page](http://health.gnu.org/) and install it using the provided script: 

```
$ wget https://ftp.gnu.org/gnu/health/gnuhealth-3.0.3.tar.gz
$ tar xvf gnuhealth-3.0.3.tar.gz
$ cd gnuhealth-3.0.3/
$ ./gnuhealth-setup install
$ source "$HOME"/.gnuhealthrc
```

By default, GNU Health Tryton server listens to the localhost on port 8000, this will not allow direct connections from other workstations. To modify this, change the configuration by typing the command `editconf`:

```
[database]
uri = postgresql://localhost:5432
path = /home/gnuhealth/attach

[jsonrpc]
listen = *:8000

[session]
super_pwd = xxxxxxxxxxxxxxxxxxxxx
```

## Create the database

Execute the command below to create a database called `gnuhealth`:

```
$ createdb --encoding=unicode gnuhealth3
```

## Start Tryton Server

To start the server, type the following commands:

```
$ cdexe
$ ./trytond --verbose
```

Create a startup script for the server by creating a file `/home/gnuhealth/start_gnuhealth.sh` with the following contents:

```shell
#!/bin/bash
source $HOME/.gnuhealthrc
cd ${GNUHEALTH_DIR}/tryton/server/${TRYTOND}/bin
python ./trytond
```

Then, create a GNU Health Unit file  `/lib/systemd/system/gnuhealth.service` :

```
[Unit]
Description=GNU Health Server
After=network.target

[Service]
Type=simple
User=gnuhealth
WorkingDirectory=/home/gnuhealth
ExecStart=/home/gnuhealth/start_gnuhealth.sh
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Start the service and enable it to start at boot time:

```
# systemctl start gnuhealth
# systemctl enable gnuhealth
```

## Install Tryton Client

The source is available for different OS and can be download from the [official website](https://downloads.tryton.org/).

Install the default modules required for the basic functionality by using these steps:

1. After creating the database, the system will ask to create some new users. This step can be skipped.
2. The next screen presents a list of modules that will provide the desired functionality. If the *Modules* window does not appear, navigate to it on the left side: *Administration → Modules → Modules*.
3. Select the **health_profile** module, and click on **Mark for installation**.
4. Clic on the **Action** icon (a blue rotated square) and select **Perform Pending Installation/Upgrade**
5. Tryton will automatically select all the dependent modules required for the installation.
6. Click on **Start Upgrade**. This process will take a while, depending on the computer where GNU 
   Health is being installed on. Once it's done, a message saying "*System upgrade done*" will appear.

## Notes

- For more details regarding the GNU Health software, please refer to the [official documentation of GNU Health](https://en.wikibooks.org/wiki/GNU_Health/Installation).
- For a quick installation (not recommended for a production use), a Docker image providing a demo of GNU Health is available [here](https://github.com/mbehrle/docker-gnuhealth-demo).

# GNU Health/FHIR REST server

Fast Healthcare Interoperability Resources (FHIR) is a standard for exchanging healthcare information electronically developed by [HL7](http://hl7.org/fhir).

Representational State Transfer (REST) is a software architecture style for scalable web services developed by the W3C Technical Architecture Group (TAG). REST based systems can be accessed over the Hypertext Transfer Protocol (HTTP), the same protocol that is used by Web browsers to request Web pages from and to send data to a Web server. For more information about REST, please see [here](https://en.wikipedia.org/wiki/Representational_state_transfer).

## Installation

The server requires Flask and a few of its add-ons and a working GNU Health installation.

It is recommended to install the packages into a virtual environment using *virtualenv*. Setup the environment as the *gnuhealth* user:

```
$ pip install virtualenv                # Install virtualenv using python (may require root)
$ cd ~/gnuhealth/fhir/server            # Wherever you want to put the virtual environment folder
$ virtualenv -p /usr/bin/python2 venv   # Create the environment, with the Python 2.x interpreter
$ source venv/bin/activate              # Active the environment
```

Now install the packages using the requirements file:

```
server/config.py
----------------
TRYTON_DATABASE = 'gnuhealth'    # GNU Health database
SERVER_NAME = '192.168.56.xx'    # Domain name of the server (e.g., fhir.example.com)
SECRET_KEY = 'BH8qXlQmdCxBBoZ3z7k2iAJxQ'  # Set this value to a long and random string
```

## Running the server

The server ships with a simple script (run_server.py) to run the server using [Tornado](http://www.tornadoweb.org/en/stable/).

With the virtual environment activated and as the *gnuhealth* user, run the server.

```
$ mv ~/gnuhealth/fhir/server/run_server.py ~/gnuhealth/fhir/run_server.py
$ cd ..
$ python run_server.py
```

However, most production servers use Nginx, Lighttpd, or Apache in front of the Tornado server. To configure an Nginx/Lighttpd + Tornado + Flask setup is beyond the scope of this document.

## Notes

- For more details regarding FHIR and REST support on GNU Health, please refer to the [official documentation](https://en.wikibooks.org/wiki/GNU_Health/FHIR_REST_server).
- Some Public FHIR Servers are available on line for testing purpose, the list can be found [here](http://wiki.hl7.org/index.php?title=Publicly_Available_FHIR_Servers_for_testing).
- Details on how to use the FHIR/REST can be found on the [official documentation of GNU Health](https://en.wikibooks.org/wiki/GNU_Health/Using_the_FHIR_REST_server).

# Contributors

- Santatra R.
- JC P.
- Jehoram M.
- Ashraf A.
- Adjeri S.