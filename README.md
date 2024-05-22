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


Therefore at first you can connect to the database and list all databases and then roles. The command to get help on commands is `\?`. 
```
$psql -U postgres -d postgres
postgres# \l
postgres#\du
postgres#\q
```
In the postgresql database you will need to create a database gnuhealth and a role gnuhealth with the password, so that this user can be used to connect to the postgresql database in order to install Tryton software.    

```

