Cloud Storage Service
=====================

Overview 
This Cloud Storage Service allows an application to integrate cloud storage into their application.  The application could then perform standard file system operations such as:
•   list
•   upload file
•   download 
•   upload folder
•   download folder
•   mkdir
•   delete
•   rename
Users can access the files via API calls (via an HTTP REST API) or via the RP-SMARF GUI.
Users can also use the RSS authorization capabilities to allow other users to access files in the cloud for data sharing.
This would allow applications to implement features such as:
•   save data for a specific user from a process run in the cloud
•   share data between users
•   share data between sites for a single user

Sample Applications
These scenarios assume that the application administrator has installed the RSS software on their own servers.
A desktop-resident application needs to implement a capability to allow users to share files. The application can suggest that its users acquire accounts and then put the end user’s email address and API key into the application.  With this in place, the tool could upload information for the user and view other users’ posted information (assuming the information can be public – if not then application can manage permissions or allow the end user to manage permissions via the RSS
UI).  Users can also upload and download the files directly via the RSS UI
A server-based application wants to allow users to upload large amounts of data for analysis by cloud-based applications.  The application can suggest that its users acquire RP-SMARF accounts and then put the end user’s email address and API key into the application.  The user can then use the RSS UI to upload the data (or other scripting-oriented tools) and then start the server-based application which will access the data by URL.

Get a Trial Account
In order to request a trial account with the RPSMARF system, send an email to support@rpsmarf.ca. Once the trial account is created, you will get an email with details regarding your owner path like /scs/user/12/ for example. You will be provided with two resources: a private cloud folder and a public cloud folder. The private cloud folder will be accessible only to you whereas the public folder will be visible to other users of the RPSMARF system. Also you will be provided with an
API Key to access the Cloud Storage API through REST calls via curl.
Sample Sequences
The sample sequences in this section demonstrate some of the few useful things you can do with the Cloud Storage API. In order to prevent providing the API key with every curl command, you may export it before trying out the sequences.
The key may be exported as:
export API_Key = API Key provided in the email
The API Key may be used in the subsequent REST calls as $API_Key.
As a first step, you can try to locate your private and public folders by browsing the resources that have you as the owner. 
curl –H “Authorization: ApiKey user_name:$API_Key” 
http://www.rpsmarf.ca/scs/resource/?user=/scs/user/12/
Once you have located your private folder and public folder you may try to make a new folder within the resource. If your public folder is /scs/resource/15/ for example, you can make a new folder named test in your public folder as follows:
curl –H “Authorization: ApiKey user_name:$API_Key” 
http://www.rpsmarf.ca/scs/resource/15/test/mkdir/
You can then do a list operation on the resource to see the newly created folder within the resource.
curl –H “Authorization: ApiKey user_name:$API_Key” 
http://www.rpsmarf.ca/scs/resource/15/list/
You may try to upload a file in the current directory to the resource as follows:
Assuming that there is a file called "x" in the working folder, you can do:

curl –H “Authorization: ApiKey user_name:$API_Key” 
-i –F uploadfile=@x http://www.rpsmarf.ca/scs/resource/15/newfile/upload/
When you do a list operation on the resource, you will see the uploaded file and the folder test.
Next, you can try to download the uploaded file as follows:
curl –H “Authorization: ApiKey user_name:$API_Key” http://www.rpsmarf.ca/scs/resource/15/newfile/download/
You may also delete the file as follows:
curl -i -H "Accept: application/json" –H “Authorization: ApiKey user_name:$API_Key”  -X DELETE http://www.rpsmarf.ca/scs/resource/15/newfile/file/
On performing a list operation on the resource, you will no longer see the file newfile.
The next section is a detailed reference guide for accessing the storage resource.


