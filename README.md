# gnuhealth_instruction
## Setup Instructions and clarification of confusing steps 
To set up and configure the open source Laboratory Information System using GNU Health one can follow the official [technical guide](https://docs.gnuhealth.org/his/techguide/installation/). 

For unexperienced user there might be several installation and configuration steps in the official guide that may be challenging and one can spend a lot of time figuring out how to resolve issues (happenned to me). Therefore, I wrote a short complement walk through the installation following a _Vanilla_ instalation and set up of GNU Health on a fresh Ubuntu 22.04.

There are two components of GNU Health - a server and a client. 

## The GNU Health server component
The GNU Health server component comprises a database (Postgresql), a Tryton enterprise software package and a GNU Health wrapper. There must be a designated user gnuhealth under whose account the GNU health service will be running. 

### Step 1. Add a user _gnuhealth_
The GNU Health requires a specialized user under whose account the system will run. By default this user has name gnuhealth. To add a user to the system you have to have sudo rights. 
```
$sudo adduser gnuhealth
```
This user will be connecting to the postgresql database and also running the gnuhealth service. You will need to suply password for this user.  

### Step 2. Database setup
The GNU Health uses PostgreSQL database, [docummentation link](https://www.postgresql.org/docs/14/index.html).

To install postgress use a standard Ubuntu package manager [(here some general tips)](https://ubuntu.com/server/docs/install-and-configure-postgresql). 
```
$sudo apt install postgresql
```
The postgresql database installs with the default role postgres and a default datbase `postgres` and a default password _postgres_. However, most likely you will see peer authentication failed for postgress. If the peer authentication is set as default you have to map a current logged to the system user to the postgres database user. The mapping file on the system is `/etc/postgresql/14/main/pg_ident.conf`. Open it with `sudo nano` and edit adding a line at the end:
```
# MAPNAME       SYSTEM-USERNAME         PG-USERNAME
mapname1        your_login_name         postgres
```
Second edit is in `/etc/postgresql/14/main/pg_hba.conf` file. There you must add the `map=mapname1` mapping to the peer authentication for postgres in a following way:

```
# Database administrative login by Unix domain socket
local   all             postgres                                peer map=mapname1
```
After these modifications you have to restart postgres server
```
sudo systemctl restart postgresql
```
Now you can connect to the database and list all databases and then roles. The command to get help on commands is `\?`. 
```
$psql -U postgres -d postgres
postgres# \l
postgres#\du
postgres#\q
```
Next create a gnuhealth user with password, you will be prompted for sudo password and then for the database user gnuhealth. For training purposes enter password _gnuhealth_. Create the database and make the gnuhealth user owner of this database as follows. List the users and databases. 
```
$psql -U postgres -d postgres
postgres# create user gnuhealth with createdb  nocreaterole nosuperuser password 'gnuhealth';
postgres# create database health owner=gnuhealth;
postgres# \l
postgres# \du
postgres# \q
```
This user and database will be used later in Tryton software installation.    

### Step 3. Install GNU health

This part is as explained in [Vanilla installation](https://docs.gnuhealth.org/his/techguide/installation/vanilla.html#initialize-the-database-instance) **Downloading and installing GNU Health**. 
Swithch to gnuhealth user. The following commands are without modification as is in Vanilla installation document. 
```
$su - gnuhealth
$cd $HOME
wget https://codeberg.org/gnuhealth/his/releases/download/v4.4.0/gnuhealth-4.4.0.tar.gz
gpg --recv-key  --keyserver  keyserver.ubuntu.com 0xC015E1AE00989199
gpg --with-fingerprint --list-keys 0xC015E1AE00989199
wget https://codeberg.org/gnuhealth/his/releases/download/v4.4.0/gnuhealth-4.4.0.tar.gz.sig
gpg --verify gnuhealth-4.4.0.tar.gz.sig gnuhealth-4.4.0.tar.gz
tar xzf gnuhealth-4.4.0.tar.gz
cd gnuhealth-4.4.0
wget -qO- https://codeberg.org/gnuhealth/his/releases/download/v4.4.0/gnuhealth-setup-4.4.0.tar.gz | tar -xzvf -
bash ./gnuhealth-setup install
source ${HOME}/.gnuhealthrc
```
This part installed all required modules and also Tryton software. Next step is to configure Tryton. 
### Step 4. Configure Tryton.
Tryton config file is `/home/gnuhealth/gnuhealth/tryton/server/config/trytond.conf`.

You will open it with the command that is simple vim editor.  
```
editconf
```
You will see the following:
[database]
uri = postgresql://localhost:5432
path = /home/gnuhealth/attach

[web]
listen = *:8000

[webdav]
listen = *:8080
ssl_webdav = False
```
Here important thing to note is that you have to provide a user and password in the [database] section to be able to connect to the postgres database to do the initial Tryton database configuration. Remember that we created a user gnuhealth with the passworg _gnuhealth_.  Edit as follows and save.  
```
[database]
uri = postgresql://gnuhealth:gnuhealth@localhost:5432
path = /home/gnuhealth/attach

[web]
listen = *:8000

[webdav]
listen = *:8080
ssl_webdav = False
```
You can set up logging as is shown in **Vanilla instalation**. We skip it here.
### Step 5. Initialization of Tryton database.
Here the tryton initializer creates empty instance of the database with all hospital information system templates. Change to newly installed system:
```
cdexe
```
Run initialization script:
```
python3 ./trytond-admin --all --database=health -vv
```
During this process you will be asked for admin email and admin password. Give pasword _admin_ so that it is easy to remember.
 
### Step 5. Create a gnuhealth service
Linux systems have very nice feature that is you can create a service that runs in the background as for example sshd or postgres or nginx services. For this you have to put a service definition files in a certain location on the file system.



   

