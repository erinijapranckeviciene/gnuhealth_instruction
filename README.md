# gnuhealth_instruction
## Setup Instructions and clarification of confusing steps 
To set up and configure the open source Laboratory Information System using GNU Health one can follow the official [technical guide](https://docs.gnuhealth.org/his/techguide/installation/). 

For unexperienced user there might be several installation and configuration steps in the official guide that may be challenging and one can spend a lot of time figuring out how to resolve issues (happenned to me). Therefore I wrote a walk through the installation and set up on Ubuntu 22.04 where appropriate giving credits to the enthusiats in various forums for sharing their knowledge and experience. 

There are two components of GNU Health - a server and a client. 

## The GNU Health server component
The GNU Health server component comprises a database (Postgresql), a Tryton enterprise software package and a GNU Health wrapper. 

### The database component


