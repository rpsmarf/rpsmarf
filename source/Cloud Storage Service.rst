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
export API_Key=”Authorization: ApiKey user_name:Key” 
where Key and user_name are provided in the email.
The API_Key may be used in the subsequent REST calls as $API_Key. 
As a first step, you can try to locate your private and public folders by browsing the resources that have you as the owner. 
curl –H “$API_Key” “http://www.rpsmarf.ca/scs/resource/?user=/scs/user/12/”
Once you have located your private folder and public folder you may try to make a new folder within the resource. If your public folder is /scs/resource/15/ for example, you can make a new folder named test in your public folder as follows:
curl –H “$API_Key” 
“http://www.rpsmarf.ca/scs/resource/15/test/mkdir/”
You can then do a list operation on the resource to see the newly created folder within the resource.
curl –H “$API_Key” 
“http://www.rpsmarf.ca/scs/resource/15/list/”
You may try to upload a file in the current directory to the resource as follows:
Assuming that there is a file called "x" in the working folder, you can do:

curl –H “$API_Key” 
-i –F uploadfile=@x “http://www.rpsmarf.ca/scs/resource/15/newfile/upload/”
When you do a list operation on the resource, you will see the uploaded file and the folder test.
Next, you can try to download the uploaded file as follows:
curl –H “$API_Key” http://www.rpsmarf.ca/scs/resource/15/newfile/download/
You may also delete the file as follows:
curl -i -H "Accept: application/json" –H “$API_Key” -X DELETE  “http://www.rpsmarf.ca/scs/resource/15/newfile/file/”
On performing a list operation on the resource, you will no longer see the file newfile.
The next section is a detailed reference guide for accessing the storage resource.


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

curl –H “$API_Key” “http://demo.rpsmarf.ca/scs/resource/3/list/”
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
                                              
curl –H “$API_Key” “http://demo.rpsmarf.ca/scs/resource/3/list/?maxEntries=1”
[
{
"basename": "101_data/",
"isDir": true,
"mtime": 1415305350.4460995,
"name": "101_data/",
"size": 0
}
]

curl –H “$API_Key” "http://demo.rpsmarf.ca/scs/resource/3/list/?maxEntries=1&lastPathReturned=101_data/"
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
                                                          
curl –H “$API_Key” “http://demo.rpsmarf.ca/scs/resource/3/2014-09-01.data/download/”
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
                                                                                {                                                                                                                                                                "path": "<path of file uploaded - if makeTempPath was true, this is how to get the path generated>"
                                                     }                                                                                                                                                                                                                                                 
**Upload Examples**
                                                                                                                                                                Assuming that there is a file called "x" in the working folder, you can do:
                                                                                                                            
curl –H “$API_Key”  -i -F uploadfile=@x http://demo.rpsmarf.ca/scs/resource/1/test/upload/

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

$ curl –H “$API_Key”  “http://demo.rpsmarf.ca/scs/resource/3/101_data/metadata/”
[
{
"isDir": true,
"mtime": 1415305350.4460995,
"name": "101_data/",
"size": 0
}
]

$ curl –H “$API_Key”  “http://demo.rpsmarf.ca/scs/resource/3/2014-09-01.data/metadata/”
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

$ curl –H “$API_Key”  http://demo.rpsmarf.ca/scs/resource/3/folder1/mkdir/

$ curl –H “$API_Key”  http://demo.rpsmarf.ca/scs/resource/3/folder2/folder3/folder4/mkdir/?makeParentFolders=True

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

curl –H “$API_Key”  “http://demo.rpsmarf.ca/scs/task/77/”

**Zip Example**
              
$ curl –H “Authorization: ApiKey user_name:$API_Key”  “http://demo.rpsmarf.ca/scs/resource/1/f1/zip/?makeTempPath=True”
{"zipFilePath": ".rpsmarf/tmp/8329c3ca-1f03-4351-8984-2fddc8a18514.zip", "taskPath": "/scs/task/1/"}

