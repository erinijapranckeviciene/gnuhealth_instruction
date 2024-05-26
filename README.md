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
```
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
Linux systems have very nice feature that is you can create a service that runs in the background as for example sshd or postgres or nginx services. For this you have to put a service definition files in a certain location on the file syst/etc/systemd/system/gnuhealth.serviceem.
In Ubuntu create a new file - use other terminal window as root-:
```
sudo nano /etc/systemd/system/gnuhealth.service
```
Enter the following content into the file and save it
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
Starting, stopping and restarting a service:
```
sudo systemctl start gnuhealth
sudo systemctl stop gnuhealth
sudo systemctl restart gnuhealth
sudo systemctl status gnuhealth
```
Enabling at boot time
```
systemctl enable gnuhealth
```
Will start the gnuhealth service later. At this time we leave ngix web server and uwsgi configuration aside and proceed to gnuhealth client installation.

## The GNU Health client component
Under the same gnuhealth user install gnuhealth client
Add PATH variable to .bashrc file
```
nano $HOME/.bashrc
#add this line
export PATH=$HOME/.local/bin:$PATH
#close the file
source $HOME/.bashrc
```
Update pip3
```
pip3 install --upgrade --user pip
```
Install gnuhealth client
```
pip3 install --user --upgrade gnuhealth-client
```
###### To troubleshoot missing paclages
As sudo install following packages:
```
sudo apt install libgirepository1.0-dev
sudo apt install python3-gi-cairo 
sudo apt install python3-gi gobject-introspection gir1.2-gtk-3.0
sudo apt install libcanberra-gtk-module libcanberra-gtk3-module
```
Under a gnuhealth user install again:
```
pip3 install PyGObject
```
## Final step: start the server and connect

Start the gnuhealth service as a root from a different terminal 
```
sudo systemctl start gnuhealth
```
Under the gnuhealth user connect to the server with gnuhealth-client
```
gnuhealth@rtx4: gnuhealth-client
```
In the connection window we must fill in host- localhost, database - health, user - admin :
![image](https://github.com/erinijapranckeviciene/gnuhealth_instruction/assets/23616522/1560afb8-e46c-4853-b005-0e229bee4a30)

Enter the admin password 'admin' and you will see the main management window:
![image](https://github.com/erinijapranckeviciene/gnuhealth_instruction/assets/23616522/ed67c4af-2f4e-44ce-a7a3-549805922133)

This is an entry to configure the system.  For futher steps use official documentation [https://docs.gnuhealth.org/his/userguide/](https://docs.gnuhealth.org/his/userguide/) . 



#### Other note about DISPLAY setuo
The gnuhealth-client is a graphical interface to the gnuhealth system. For it to run we need to configure graphical interface _or try through web interface_.  [Configuring DISPLAY](https://unix.stackexchange.com/questions/613458/how-to-enable-xhost-access-from-second-user-when-display0-is-on-first-user)

On Virtual machines this trick will not work. We need gnuhealth to connect via ssh -X :
```
$ssh -X gnuhealth@xxx.xxx.xxx.xxx
```
Or we need to configure web interface access. 

## The GNU Health web client setup intructions on localhost
GNU Health system can be accessed by the tryton web client. Unfortunately the docummentation on how to install trytond sao webclient in various places is not complete. Here is a summary of steps that allow to start sao webclient. At first it is installed on localhost:8000. 

### Step 1. Installing Node.js and node package manager npm under the local user

Change to the gnuhealth user
```
$su - gnuhealth
```
Install Node.js and npm using node version manager nvm. The dependencies are written to .bashrc. Everytime you need to work with npm you must source the .bashrc .
```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
$ nvm install node
$source $HOME/.bashrc
$node --version
$npm --version
```
Install other packages that may be needed , under the local user 
```
$npm install -g grunt-cli
$npm install grunt
```
In addition note,  if a specific version of Node.js is needed, then you can install it as for example: `nvm install v14.16.1`. 

### Step 2. Obtain tryton web client SAO and install it.
Continue as user gnihealth
```
$cd $HOME
$mkdir sao

# Get the sao software from tryton official site. Our version is 6.0. Get the latest package.
$wget https://downloads.tryton.org/6.0/tryton-sao-last.tgz

#extract. It extracts the package dir, enter the package dir
$tar -xvf tryton-sao-last.tgz
$cd package

#install sao using npm
$npm install --production --legacy-peer-deps
```
### Step 3. Add necessary information into tryton config file
```
$nano $HOME/gnuhealth/tryton/server/config/trytond.conf
```
The [web] and [jasonrpc] must contain the path to the sao folder:
```
[database]
uri = postgresql://trydbuser:trydbuser@localhost:5432
path = /home/gnuhealth/attach

[web]
listen = localhost:8000
root = /home/gnuhealth/sao/package

[jsonrpc]
data= /home/gnuhealth/sao/package
listen = *:8000

[webdav]
listen = *:8080
ssl_webdav = False
```
### Step 4. Invoke the webclient
Open your webbroser and enter http://localhost:8000 . You must receive the authentication window where you enter the database user admin with password _admin_ . This is the user that you created upon database initialization.
![image](https://github.com/erinijapranckeviciene/gnuhealth_instruction/assets/23616522/4db8cdd4-355f-46e8-b0ad-72daea407d0b)

After you log in you will see a database management window like this:

![image](https://github.com/erinijapranckeviciene/gnuhealth_instruction/assets/23616522/0d234784-79c3-4064-a7c2-dca71c9fe3f4)


From this point forward - the module instalation and configuration must be well described in GNU Health docummentation. 

















   

