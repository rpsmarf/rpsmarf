Cloud Storage Service
=====================

Overview
--------
This Cloud Storage Service allows an application to integrate cloud storage into their application.  The application could then perform standard file system operations such as list, upload file, download file, upload folder, download folder, make a directory, delete and rename.
Users can access the files via API calls (via an HTTP REST API). Users can also use the authorization capabilities to allow other users to access files in the cloud for data sharing.

This would allow applications to implement features such as:

* save data for a specific user from a process run in the cloud
* share data between users
* share data between sites for a single user

Sample Applications
-------------------
A desktop-resident application needs to implement a capability to allow users to share files. The application can suggest that its users acquire accounts and then put the end user’s email address and API key into the application.  With this in place, the tool could upload information for the user and view other users’ posted information (assuming the information can be public).

A server-based application wants to allow users to upload large amounts of data for analysis by cloud-based applications.  The application can suggest that its users acquire RP-SMARF accounts and then put the end user’s email address and API key into the application.  The user can then upload the data and then start the server-based application which will access the data by URL.


Get a Trial Account
-------------------
In order to request a trial account with the RPSMARF system, send an email to support@rpsmarf.ca. Once the trial account is created, you will get an email with details regarding your owner path like /scs/user/12/ for example. You will be provided with two resources: a private cloud folder and a public cloud folder. The private cloud folder will be accessible only to you whereas the public folder will be visible to other users of the RPSMARF system. Also you will be provided with an API Key to access the Cloud Storage API through REST calls via curl.

Sample Sequences
----------------
The sample sequences in this section demonstrate some of the few useful things you can do with the Cloud Storage API. In order to prevent providing the API key in the header with every curl command, you may export it before trying out the sequences.

The key may be exported as:

export API_Key="Authorization: ApiKey user_name:Key" 

Please note that key and user_name will be provided in the email.

The API_Key may be used in the subsequent REST calls as $API_Key.

As a first step, you can try to locate your private and public folders by browsing the resources that have you as the owner.

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/resource/?user=/scs/user/12/"

Once you have located your private folder and public folder you may try to make a new folder within the resource. If your public folder is /scs/resource/15/ for example, you can make a new folder named test in your public folder as follows:

curl –H "$API_Key" "https://www.rpsmarf.ca/scs/resource/15/test/mkdir/"

You can then do a list operation on the resource to see the newly created folder within the resource.

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/resource/15/list/"

You may try to upload a file in the current directory to the resource as follows:

Assuming that there is a file called "x" in the working folder, you can do:

$ curl –H "$API_Key" -i –F uploadfile=@x "https://www.rpsmarf.ca/scs/resource/15/newfile/upload/"

When you do a list operation on the resource, you will see the uploaded file and the folder test.

Next, you can try to download the uploaded file as follows:

$ curl –H "$API_Key" "https://www.rpsmarf.ca/scs/resource/15/newfile/download/"

You may also delete the file as follows:

$ curl -i -H "Accept: application/json" –H "$API_Key" -X DELETE  "http://www.rpsmarf.ca/scs/resource/15/newfile/file/"

On performing a list operation on the resource, you will no longer see the file newfile.

The next section is a detailed reference guide for accessing the storage resource.


Reference Guide
---------------
This section describes in detail the various operations that may be performed on files and folders within the cloud storage.

File operations on files accessible within RP_SMARF may be accessed via the REST API. Files and folders are accessible under the "resource" object which maps to the folder which contains the file or folder to be operated on.

The general form for a URL is:

<resource path>/<file/folder path>/<operation>/<parameters, if any>

The elements in the URL are as described:

**Resource Path**

This is the path which associates the request with the folder on the associated remote agent, /scs/resource/15 for example.

**File/Folder Path**

This is the path within the folder associated with the resource that the request operates on.

**Operation**

As mentioned in the overview, the operation may be one of the following:

* Download (download)
* Upload (upload)
* List (list)
* Metadata (metadata)
* Make a Directory (mkdir)
* Zip (zip)
* Unzip (unzip)

If an operation succeeds no data is returned. If a request fails for a syntax reason (e.g. missing argument) then a 400 HTTP code will be returned. If a request fails for a semantic reason (e.g. file not exists) then a 422 will be returned. If a 400 or 422 is returned then the following JSON structure is returned:

  {
  "code": "<short code>",
  "message": "<detailed message>"
  }
  For example:
  {
  "code": "PERM",
  "message": "The file was not deleted because the file system did not allow it"
  }

Each of these operations are described below in detail.

**List**

  The list operation lists all the contents (files and folders) of a resource.

  The various parameters of the list operation are:

1. maxEntries: This parameter specifies the maximum number of entries to return. If there are more entries, the list operation can be repeated with the lastPathReturned parameter set to the name of the last path returned. This parameter is of type int and its default value is 1000.

2. recursiveDepth: This parameter specifies the maximum depth to walk when listing a folder. A value of 1 lists only the current folder. This parameter is of type int and its default value is 1.

3. lastPathReturned: This parameter specifies the next path to return in a 'paged' listing of files. This is done by setting this parameter to the path of the last file/folder returned and the list operation will return entries which occur after the path specified by this parameter. This parameter is of type string and its default value is "".

**List Examples**

$ curl –H "$API_Key" "http://demo.rpsmarf.ca/scs/resource/3/list/"
[
{
"basename": "101_data/",
"isDir": true,
"mtime": 1415305350.4460995,
"name": "101_data/",
"size": 0
},
{
"basename": "2014-09-01.data",
"isDir": false,
"mtime": 1414519068.0,
"name": "2014-09-01.data",
"size": 28
},
{
"basename": "2014-09-02.data",
"isDir": false,
"mtime": 1414519068.0,
"name": "2014-09-02.data",
"size": 28
},
...

Listing files, one by one:
                                              
$ curl –H "$API_Key" "http://www.rpsmarf.ca/scs/resource/3/list/?maxEntries=1"
[
{
"basename": "101_data/",
"isDir": true,
"mtime": 1415305350.4460995,
"name": "101_data/",
"size": 0
}
]

$ curl –H "$API_Key" "http://demo.rpsmarf.ca/scs/resource/3/list/?maxEntries=1&lastPathReturned=101_data/"
[
{
"basename": "2014-09-01.data",
"isDir": false,
"mtime": 1414519068.0,
"name": "2014-09-01.data",
"size": 28
}
]

**Download**

The download operation downloads a file within the resource.
                                                          
**Download Example**
                                                          
$ curl –H "$API_Key" "http://demo.rpsmarf.ca/scs/resource/3/2014-09-01.data/download/"
{ 
"segmentSize" : 0.04    <-- Actual file contents
}

**Upload**

The upload operation uploads a file within the resource.
                                                           
**Upload Parameters**

The various parameters of the upload operation are:

1. makeTempPath: This parameter specifies that the path in the request will be ignored and a path will be generated. This is returned in the path element in the data returned. This parameter is of the type bool and its default value is False.
                                                                                                                           
2. Overwrite: If False then if the file path specified already exists, the upload operation fails. If True, then if the file exists, that file is atomically replaced with the uploaded file. The default is False. 
   
The return data for upload is as follows:

{                                     
 "path": "<path of file uploaded - if makeTempPath was true, this is how to get the path generated>"                                                     }                                                                                                                                                                                                                                                 
**Upload Examples**
                                                                                                                                                                Assuming that there is a file called "x" in the working folder, you can do:
                                                                                                                            
$ curl –H "$API_Key"  -i -F uploadfile=@x "http://demo.rpsmarf.ca/scs/resource/1/test/upload/"

HTTP/1.1 100 Continue
HTTP/1.1 200 OK
Server: nginx/1.6.2
Date: Wed, 10 Dec 2014 15:42:39 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Frame-Options: SAMEORIGIN
{"path": "test"}

**Metadata** 

The metadata operation enables you to see metadata of a file or a folder within the resource.

**Metadata Examples**

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/3/101_data/metadata/"
[
{
"isDir": true,
"mtime": 1415305350.4460995,
"name": "101_data/",
"size": 0
}
]

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/3/2014-09-01.data/metadata/"
[
{
"isDir": false,
"mtime": 1414519068.0,
"name": "2014-09-01.data",
"size": 28
}
]

**Mkdir**

The mkdir operation enables you to make a folder/directory within a resource.

**Mkdir Parameters**

The various parameters of the mkdir operation are:

1. makeParentFolders: This parameter is of type bool and its default value is False. If true, then any missing parent folders are created. This is equivalent to the -p option on mkdir in Unix.

**Mkdir Examples**

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/3/folder1/mkdir/"

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/3/folder2/folder3/folder4/mkdir/?makeParentFolders=True"

**Zip**

The zip operation enables you zip a file or a folder.

**Zip Parameters**

The various parameters of the zip operation are:

1. zipFilePath: This is the path to the location of the file to be created by zipping the contents in the path of the request. This is not required in the parameter makeTempPath is set to True. It is of the type string and the default value is “”.

2. makeTempPath: This parameter specifies that the system should generate the filename for the zip file to be created. This parameter is of the type bool and the default value is False.

**Return Data**
     {
     "zipFilePath": "<path of zip file>",
     "taskPath": "<path to task running the zip operation, e.g.: /scs/task/77>"
     }

Any operation which can take a long time to complete needs to have a mechanism to report the progress of the operation. Zip is an example of one of these operations. Other such operations are recursive delete of a folder and unzip.
              
For operations which can take a long time a task is created internally within the system and then return the path of the task is returned to allow the caller to poll the state of the task to observe progress and whether the operation eventually completes successfully.
              
The progress of the task can be seen by issuing the curl command as follows:

curl –H "$API_Key" "http://demo.rpsmarf.ca/scs/task/77/"

**Zip Example**
              
$ curl –H "Authorization: ApiKey user_name:$API_Key"  "http://demo.rpsmarf.ca/scs/resource/1/f1/zip/?makeTempPath=True"
{"zipFilePath": ".rpsmarf/tmp/8329c3ca-1f03-4351-8984-2fddc8a18514.zip", "taskPath": "/scs/task/1/"}

**Unzip**

The unzip operation enables you to unzip a zipped file.

**Unzip Parameters**

The various parameters of the zip operation are:

1. zipFilePath: This is the path of the file to unzip into the folder specified in the request. This parameter is of the type string and its default value is “”.

Return Data:
{
"taskPath": "<path to task running the unzip operation, e.g.: /scs/task/77>"
}

**Unzip Example**

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/1/f1/unzip/?zipFilePath=.rpsmarf/tmp/e8970d5d-3fef-4bcb-9637-9b4c05949f27.zip"

{"taskPath": "/scs/task/2/"}

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/task/2/"      
{
"completion": "completedWithoutError",
...
"completionReason": "success",
...
"state": "finished",
...
}

**Rename**

The rename operation enables you to rename a file in a specific folder. The file stays in the same folder.

**Rename Parameters**

The parameters for the rename operation are:

1. newName: This is the new name of the file whose path is specified. This parameter is of the type string.
   
**Return Data**
{

}

**Rename Example**

$curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/1/f1/rename/?newName=f2"

{}

**Move**

This API moves a file in a folder to another folder, possibly with a new name.

**Move Parameters**

The parameters for the Move operation are:

1. newPath: This is the new path of the file whose path is specified.

** Return Data**
{

}

**Move Example**

$ curl –H "$API_Key"  "http://demo.rpsmarf.ca/scs/resource/1/a/b/f1/move/?newPath=a/c/f2"

{}

**Delete Single File or Folder Operation**

This call deletes a single file or folder (which must be empty).

The path after the resource path contains the path to the file or folder to delete followed by /file/ if the path is for a file or /folder/ if the path is a folder.

**Delete Example**

To delete the folder xx within /scs/resource/1/

$ curl -i -H "Accept: application/json" –H "$API_Key" –X DELETE "http://localhost/scs/resource/1/xx/folder/"

HTTP/1.1 204 NO CONTENT
...

**Example of error (deleting a file as a folder)**

curl-i -H "Accept: application/json" –H "$API_Key" -X DELETE "http://localhost/scs/resource/1/z/folder/"

HTTP/1.1 422 OK
...
{"osPath": "/tmp/src/z/", "description": "Not a directory", "code": "ENOTDIR", "apiPathName": "z/", "operation": "delete"}
 
**Delete Recursive Operation**

This function starts a task which will recursively delete the folder specified.

There are no parameters beside the path.

**Delete Recursive Example**

$ curl -v –H "$API_Key" "http://demo.rpsmarf.ca/scs/resource/1/a/b/f1/deleterecursive/"

{"taskPath": "/scs/task/2/"}

$ curl –H "$API_Key" "http://demo.rpsmarf.ca/scs/task/2/"

{
"completion": "completedWithoutError",
...
"completionReason": "success",
...
"state": "finished",
...
}

**Set User Permissions to Access a Resource**

The setperm command sets the permissions to access a resource.

**Set User Permissions Parameters**

The various parameters of the setperm command are:

1. user: This is the path of the user (e.g. /scs/user/1/) whose permissions are being altered. This parameter is of the type string.

2. action: This parameter is of the type string and can take the values assign which sets the permission(s) defined or remove which deletes the permission(s) specified.

3. perm: This parameter is also of the type string and can be any combination (in any order) of "r", "w" and "x" where “r” indicates read permission, “w” indicates write permission and “x” indicates execute (aka run) permission.

**Set User Permission Example**

$ curl -H "$API-Key" "http://demo.rpsmarf.ca/scs/resource/1/setperm/?user=/scs/user/1/&perm=rw&action=assign"

**Get User Permissions to Access a Resource**

The getperm command gets the permissions users have to access a resource.

**Get User Permission Parameters**

There are no parameters for the getperm request.
**Get User Permission Example**

curl -H "$API-Key" "http://demo.rpsmarf.ca/scs/resource/1/getperm/"

{
"/scs/user/1/": [
"add_smmodelresource",
"change_smmodelresource",
"delete_smmodelresource",
"read_resource_data",
"execute_on_resource",
"write_resource_data"
],
"/scs/user/2/": [
"execute_on_resource"
]
}

Deploying Services
------------------

This section describes how you can log in to the DAIR cloud, set up a target VM, use the VM to run the RPSMARF software on it and then set up a new remote agent and a container.

**Request Access to the DAIR Cloud Server**

In order to log in to the DAIR cloud server, please go to http://fluidsurveys.com/s/DAIRsubmission/ and submit a request.
Once you have received the credentials to log in to the DAIR cloud server go to https://nova-ab.dair-atir.canarie.ca/ and log in with your credentials.

**Create VMs in DAIR**

This section describes how to create a VM in the DAIR cloud.

1. Create a key pair.

    a. Accessed through "Access & Security" -> "Keypairs".  When a key pair is created you will download a .pem file. This file will be used to access the VM through ssh. Keep this private!
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
    c.  In the modal window that pops up input the details. In the "Details" section input information such as name and the type of VM to run the image on. Additionally you can select from a selection of images.
    d.  For "Image" select Ubuntu 12.04. 
    e.  In the "Access and Security" section select your key pair (very important!). 
    f.  If needed add details for "Volume Options" and "Post-creation". Volume options allows you to launch with attached storage. Post-creation allows you to upload scripts that will run once the VM boots. Can be used for some configuration.
    g.  NOTE: if you created an instance from one of the default images

4.  Assign an IP address to your Instance.

    a.   Accessed through "Access and Security" -> "Floating IPs". 
    b.  Click "Allocate IP to Project". Select pool as "nova" and click "Allocate IP". 
    c.  Select "Instances" on left hand dashboard nav. 
    d.  For the desired instance Click "Associate Floating IP", in the modal window select the Instance and the IP that you wish to assign to the selected instance. 
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
2.  Set the region to quebec or alberta via the pull-down selection in the upper right-hand region - note quebec should be faster for most purposes.
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

     a.   curl -i -H "Content-Type: application/json" –H “$API_Key”-d '{"owner": "/scs/user/1/", "name": "<name>", "guid":"<name of remote agent in sra.conf>","agentUrl":"ice://<hostname of remote agent>"} ' "http://localhost/scs/agent/"  
     b.  Check the result and status after 15 seconds or so and we should see the status go to "up" as the remote agent registers with the SCS (assuming it is running):

     $ curl -H "$API-Key" "http://demo/scs/agent/2/"  <-- Use value returned in step a 

**Setting up a Container**

Next we create a container which refers to the agent just created.

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{ "name": "Alberta Container", "name_key": "testab1_tmp", "containerUrl":"local://localhost/tmp", "agent": "/scs/agent/2/"} ' "http://demo.rpsmarf.ca/scs/container/"

After setting up the container, you can create your own storage resources within the container.

**Resource Types and Resources**

You may create a new resource type by doing the following. Please note that storage resources have the type data.

$ curl -i -H "Content-Type: application/json" –H "$API_Key" -d '{"name": "tool_copy_source", "nature": "data", "description": "This is a source of copy data"}' "http://demo.rpsmarf.ca/scs/resource_type/"

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

$ curl -i -H "Content-Type: application/json" -H "$API_Key" -d '{"name": "data_repo","resource_type": "/scs/resource_type/1/", "container": "/scs/container/1/", "owner": "/scs/user/1/"}' "http://demo.rpsmarf.ca/scs/resource/"

HTTP/1.1 201 CREATED
Server: nginx/1.6.2
Date: Fri, 21 Nov 2014 18:58:16 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Location: http://demo.rpsmarf.ca/scs/resource/1/
Vary: Accept
X-Frame-Options: SAMEORIGIN


