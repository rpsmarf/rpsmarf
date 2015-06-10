Metadata Search Service
=======================

Overview 
--------

The Metadata Search API feature allows an application to upload metadata from their application. The users can then issue queries via the REST API and then use the resulting list of matching URLs within the invoking application.

Metadata is "data about data".  Users can add metadata about files contained in resources and then search the metadata for files which match specified criteria.

For example, within a set of bridge monitoring files there is information about wind speed and wind direction.  A metadata extraction process will extract the metadata for each file and upload the metadata to the metadata repository. When a researcher wants to examine the behaviour of the structure when the wind speed is over 50 km/h from the south-east to south-west direction (i.e. 135 to 225 degrees from north), he can perform a search which will examine the extracted metadata and show the researcher a list of files which match these criteria.  The researcher can then perform analysis with these data files.

Sample Applications
-------------------

A desktop-resident application needs to implement sophisticated data searching. The application can suggest that its users acquire accounts and then put the end user’s email address and API key into the application.  With this in place, the tool could extract metadata from files on the PC and upload this metadata to the server.  The application could then perform a search by submitting the user-specified search parameters to the server and then allow the user to browse the results.

A domain-specific server-based application which stores user data in the server or elsewhere extracts metadata from the files stored in the network and uploads this metadata to the servers.  Users could perform searches and download data which matches the search criteria. 

Get a Trial Account
-------------------

If you are not already an RPSMARF system user, you can request a trial account with the RPSMARF system by sending an email to support@rpsmarf.ca. Once the trial account is created, you will get an email with details regarding your owner path like /scs/user/12/ for example. You will also be provided two resources: a private cloud folder and a public cloud folder. The private cloud folder will be accessible only to you whereas the public folder will be visible to other users of the RPSMARF system. Also you will be provided with an API Key to access the Metadata Search API through REST calls via curl. 

Reference Guide/Sample Sequences
--------------------------------

The sample sequences in this section demonstrate some of the few useful things you can do with the Metadata Search API. In order to prevent providing the API key in the header with every curl command, you may export it before trying out the sequences.

The key may be exported as:

export API_Key="Authorization: ApiKey user_name:Key" 
where Key and user_name are provided in the email.

The API_Key may be used in the subsequent REST calls as $API_Key. 

As mentioned in the previous section, you will be provided two resources, a private cloud folder and a public cloud folder when requesting a trial account. These folders are storage resources and may be accessed as follows:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/resource/"

Where www.rpsmarf.ca is the node running the Smarf Control Server Software.

This returns the list of all the resources in the system. You will be able to see the resources corresponding to your private and public folder say /scs/resource/1/ and /scs/resource/2/ for example.

Please note that each resource has a resource type, storage or compute for example. The storage resources have been defined to be of nature data. Please see the resource type corresponding to your private and public cloud folder (/scs/resource_type/1/ and /scs/resource_type/2/ for example)

You may create a new resource type by doing the following. Please note that before creating a resource you need to create a resource type for the resource. 

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"name": "tool_copy_source", "nature": "data", "description": "This is a source of copy data"}' 
"https://www.rpsmarf.ca/scs/resource_type/"

HTTP/1.1 201 CREATED
Server: nginx/1.6.2
Date: Fri, 21 Nov 2014 18:05:04 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Location: https://www.rpsmarf.ca/scs/resource_type/1/
Vary: Accept
X-Frame-Options: SAMEORIGIN

A new resource may be now be created as following. The resource type is /scs/resource_type/1/. A container object refers to the location of the resource folder on the host machine.

$ curl -i -H "Content-Type: application/json" -H "$API_Key" -d '{"name": "data_repo","resource_type": "/scs/resource_type/1/", "container": "/scs/container/1/", "owner": "/scs/user/1/"}' "https://www.rpsmarf.ca/scs/resource/"

HTTP/1.1 201 CREATED
Server: nginx/1.6.2
Date: Fri, 21 Nov 2014 18:58:16 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Location: https://www.rpsmarf.ca/scs/resource/1/
Vary: Accept
X-Frame-Options: SAMEORIGIN

The Metadata Search API consists of the following operations:

**Defining the Metadata Schema**

The first step for new users of metadata is to define a metadata schema (md_schema) which consists of a description and a set of metadata property type (md_property_type) objects.

To define a schema, do:

$ curl -i -H "Content-Type: application/json" -H "$API_Key" -d '{"resource_type": "/scs/resource_type/1/", "name": "Bridge Data", "community": "/scs/community/1/"}' "https://www.rpsmarf.ca/scs/md_schema/"

HTTP/1.0 201 CREATED
Date: Wed, 03 Jun 2015 19:53:12 GMT
Server: WSGIServer/0.2 CPython/3.4.1
Vary: Accept
Location: https://www.rpsmarf.ca/scs/md_schema/4/
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN

To see a description of each field in the md_schema object do:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/md_schema/schema/"

To add a property type do:

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"name": "Wind Speed", "type": "INT", "schema": "/scs/md_schema/4/"}' "https://www.rpsmarf.ca/scs/md_property_type/"

HTTP/1.0 201 CREATED
Date: Wed, 03 Jun 2015 19:57:26 GMT
Server: WSGIServer/0.2 CPython/3.4.1
Vary: Accept
Location: https://www.rpsmarf.ca/scs/md_property_type/1/
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN

To see a description of each field in the md_property_type object do:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/md_property_type/schema/"

**Setting up the Metadata Repository**

Once the metadata schema has been setup one or more metadata repositories can be created.  Note that a researcher can have any number of metadata repositories on the same data.  This allows researchers to have a repository that they are working with while they are developing and debugging new metadata extraction tools that put data into a different metadata repository.

To add a metadata repository:

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"resources": ["/scs/resource/1/"], "name": "Bridge Data Repo 1", "schema": "/scs/md_schema/1/"}' "https://www.rpsmarf.ca/scs/md_repo/"

HTTP/1.0 201 CREATED
Date: Wed, 03 Jun 2015 20:15:31 GMT
Server: WSGIServer/0.2 CPython/3.4.1
Vary: Accept
X-Frame-Options: SAMEORIGIN
Content-Type: text/html; charset=utf-8
Location: https://www.rpsmarf.ca/scs/md_repo/1/

To see a description of each field in the md_repo object do:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/md_repo/schema/"

**Populating the Metadata Repository with Metadata**

Metadata in the system is represented by two types of records:

* md_path records which represent files
* md_property records which represent individual properties

To add a path record:

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"relative_path": "a/b", "resource": "/scs/resource/1/", "repo": "/scs/md_repo/1/"}' "https://www.rpsmarf.ca/scs/md_path/

 HTTP/1.0 201 CREATED
 Date: Wed, 03 Jun 2015 21:28:16 GMT
 Server: WSGIServer/0.2 CPython/3.4.1
 Content-Type: text/html; charset=utf-8
 Location: https://www.rpsmarf.ca/scs/md_path/3/
 Vary: Accept
 X-Frame-Options: SAMEORIGIN
 
To see a description of each field in the md_path object do:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/md_path/schema/" 

To see a description of each field in the md_property object do:

$ curl –H "$API_Key" "https://demo.rpsmarf.ca/scs/md_property/schema/"
 
To add a property record:
 
$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"value": "33", "property_type": "/scs/md_property_type/1/", "path": "/scs/md_path/1/", "repo": "/scs/md_repo/1/"}' "https://www.rpsmarf.ca/scs/md_property/"
 
 HTTP/1.0 201 CREATED
 Date: Wed, 03 Jun 2015 21:30:37 GMT
 Server: WSGIServer/0.2 CPython/3.4.1
 Content-Type: text/html; charset=utf-8
 Location: https://www.rpsmarf.ca/scs/md_property/5/
 Vary: Accept
 X-Frame-Options: SAMEORIGIN

**Performing Searches**

The search operation returns a list of md_path records (i.e. records with a resource and a relative_path) the search command operates on the md_path object.

The URL syntax is:
 
<host and port>/scs/md_path/search?repo=<repo id>&<prop type name>[__<operator>]=<prop value>&<prop type name>[__<operator>]=<prop value>...
 
The various parameters of the search operation are:

 1. repo: This parameter is the id of the repo, 1 for example.
 2. prop_type_name: This is the name of the property that the user is interested in.
 3. operator: The records returned depend upon the operator.
 4. prop_value: This is the value of the property. 

Deploying Services
------------------

This section describes how you can log in to the DAIR cloud, set up a target VM, use the VM to run the RPSMARF software on it and then set up a new remote agent and a container.  

**Request Access to the DAIR Cloud Server**
     
In order to log in to the DAIR cloud server, please go to http://fluidsurveys.com/s/DAIRsubmission/ and submit a request.

Once you have received the credentials to log in to the DAIR cloud server go to https://nova-ab.dair-atir.canarie.ca/ and log in with your        credentials.
       
**Create VMs in DAIR**
        
This section describes how to create a VM in the DAIR cloud.
         
1. Create a key pair.
                      
  a. Accessed through "Access & Security" -> "Keypairs".  When a key pair is created you will download a .pem file. This file will be used to    access the VM through ssh. Keep this private!
  b. Click 'Create Keypair".
  c. Enter in the name of your key pair.
  d. Click "Create Keypair" in the modal window.

2. Add rules to the default security group or add security group with rules. Rules specify which ports are opened (all closed by default).
    
  a. Accessed through "Access & Security" -> "Security Groups".
  b. Adding SSH:

    i.   On "default" security group click "Edit Rules".
    ii.  Click "Add Rule". Modal window will pop up.
    iii. For "Protocol" select "TCP".
    iv.  For "Open" select "Port".
    v.   For "Port" enter "22".
    vi.  For "Source" select "CIDR".
    vii. For "CIDR" enter IP Addresses allowed to access the instance. Default of 0.0.0.0/0 will allows all addresses.
    viii.   Click "Add".

3. Launch an instance.

  a.  Accessed through "Instances".
  b.  Click "Launch Instance".
  c.  In the modal window that pops up input the details. In the "Details" section input information such as name and the type of VM to run the  image on. Additionally you can select from a selection of images.
  d.  For "Image" select Ubuntu 12.04.
  e.  In the "Access and Security" section select your key pair (very important!).
  f.  If needed add details for "Volume Options" and "Post-creation". Volume options allows you to launch with attached storage. Post-creation   allows you to upload scripts that will run once the VM boots. Can be used for some configuration.
  g.  NOTE: if you created an instance from one of the default images

4.  Assign an IP address to your Instance.
     
  a.   Accessed through "Access and Security" -> "Floating IPs".
  b.  Click "Allocate IP to Project". Select pool as "nova" and click "Allocate IP".
  c.  Select "Instances" on left hand dashboard nav.
  d.  For the desired instance Click "Associate Floating IP", in the modal window select the Instance and the IP that you wish to assign to the  selected instance.
  e.  Click "Associate".
                              
5.  Accessing Instance through SSH (from a Linux machine).
                                                                 
  a.  Open command prompt.
  b.  Enter "ssh -i /path/to/keypair.pem ubuntu@ip.address"
  c.  /path/to/keypair.pem should be the path to your downloaded .pem file (key pair)
  d.  ip.address is the floating ip address you gave to your instance.
  e.  ubuntu is the default user when creating an ubuntu instance.

**How to use a VM in DAIR and run the Service Software on it**
 
 This section describes how to take a vanilla Ubuntu 14.04 VM in DAIR and run the SMARF software on it.
  
1.  Log in to the DAIR system
2.  Set the region to quebec or alberta via the pull-down selection in the upper right-hand region - note quebec should be faster for most       purposes.                       
3.  Create a VM in DAIR  by doing:

    a.  Select Launch Instance by pressing the button on top right
    b.  Set Instance Boot Source to Snapshot
    c.  Set the Instance Snapshot to rpsmarf_ubuntu_14_04
    d.  Set Instance Name to <tag>.rpsmarf.ca
    e.  Set the Flavor to m1.small
    f.  Click Launch
    g.  If required allocate a floating IP address by

        i.  waiting until the node is up
        ii. clicking on Access and Security on the left hand side and selecting the Floating IPs tab. Click on Allocate IPs to Project.

    h.  Wait for the node to boot and then under the More button choose Associate Floating IP and associate an IP address.
    i.  If required,go to GoDaddy and set up a name for this IP address.

4.  SSH to the node by doing sshto <tag> e.g. sshto test to access test.rpsmarf.ca in your Ubuntu development VM

**Setting Up a New Remote Agent**
 
This section describes how to setup a new remote agent and get it connected to the SCS that will control it.
  
1.  SSH to the newly created node
2.
          
  a.  sudo emacs /etc/hosts and add:
  b.  <private IP address of the SCS node> <hostname of the SCS node>
  c.  For example:
  d.  10.0.82.6 demo.rpsmarf.ca
                        
3.  Install SRA
                                                     
  a.  sudo apt-get install smarf-sra
                              
4.  sudo emacs /etc/smarf-sra/sra.conf and add:

     a.  export AGENT_GUID=<name of remote agent>      <-- note this name must match the name in the agent object created in the steps below
     b.  export SCS_HOST_NAME=<host name of the Control Server> (e.g. demo.rpsmarf.ca)

5.  sudo service smarf-sra restart
6.  Exit the shell
7.  SSH to the SCS node
8.  Create the agent object using the REST API.

  a.   $ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"owner": "/scs/user/1/", "name": "<name>", "guid":"<name of remote agent   in sra.conf>","agentUrl":"ice://<hostname of remote agent>"} ' "http://localhost/scs/agent/"
  
  b.  Check the result and status after 15 seconds or so and we should see the status go to "up" as the remote agent registers with the SCS     (assuming it is running):
        
        $ curl -H "$API-Key" "http://demo/scs/agent/2/"  <-- Use value returned in step a

**Setting up a Container**
         
Next we create a container which refers to the agent just created.
          
$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{ "name": "Alberta Container", "name_key": "testab1_tmp", "containerUrl":"local:// localhost/tmp", "agent": "/scs/agent/2/"} ' "http://demo.rpsmarf.ca/scs/container/"
           
After setting up the container, you can create your own storage resources within the container.
            
**Resource Types and Resources**
             
You may create a new resource type by doing the following. Please note that storage resources have the type data.
              
$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"name": "tool_copy_source", "nature": "data", "description": "This is a source   of copy data"}' "http://demo.rpsmarf.ca/scs/resource_type/"
               
Please note that storage resources have the type data.
                
HTTP/1.1 201 CREATED
Server: nginx/1.6.2
Date: Fri, 21 Nov 2014 18:05:04 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Location: http://demo.rpsmarf.ca/scs/resource_type/1/
Vary: Accept
X-Frame-Options: SAMEORIGIN

A new resource may be created as following. The following example assumes that the resource type is /scs/resource_type/1/.
 
$ curl -i -H "Content-Type: application/json" -H "$API_Key" -d '{"name": "data_repo","resource_type": "/scs/resource_type/1/", "container": "/scs/ container/1/", "owner": "/scs/user/1/"}' "http://demo.rpsmarf.ca/scs/resource/"
  
HTTP/1.1 201 CREATED
Server: nginx/1.6.2
Date: Fri, 21 Nov 2014 18:58:16 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Location: http://demo.rpsmarf.ca/scs/resource/1/
Vary: Accept
X-Frame-Options: SAMEORIGIN

**How to Configure the VM in the Most Basic Configuration**

1.  To initialize the control server, run the script scsInstallAndRestart.sh.
2.  To initialize the remote agent
3.  Wipe and initialize the database by running the script scsWipeAndInit.py
4.  You can set up the VM in the basic single node configuration by running the script  /opt/smarf-scs/scs/setup/singleNode/scs_core_setup.py.

