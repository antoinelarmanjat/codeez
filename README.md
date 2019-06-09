Codeez is a package to access all codeez features.

It consists of two separate classes: codeez itself and micro-service

Codeez itself allows you to access all codeez functions on your account:
- create a microservice or a website
- access the list of micro-services and websites, whether they are yours, your team's or the community's
- fork, delete a website or microservice
- start or stop a microservice
- store directory data
- access the database of a microservice in case there is one

All classes use the codeez apis. These apis are also available directly at https://api.codeez.tech

to initialize a codeez session, provide the user id and the user authorization API key as follows:

import codeez

mycodeez = codeez.codeez('user','userauthorization') 
where user is the user code and userauthorization is the user token that is provided at user creation time

In the codeez class, here are the following functions:

init: is used when thre codeez class is instantiated. Two parameters are set at that stage: User is your userID and Authorization is your APIkey.

create_user: (self, email, password, FirstName, LastName, AccountAuthorization).
To create a new user, you need to provide email, password (with at least upper, lower, number and special character), first name and last name and accountauthorization.
The account authorization number is provided by codeez by your administrator when your account is created. Once validated the account administrator recieves a code that is required when creating a user.

request_account(self, email, GroupName, Description):
This is when an admnistrator requests the creation of an account. This goes in a queue and is managed by the codeez administrators to review the request. When deemed appropriate, the account is validated.
Once validated by codeez, your will receive the account code that is required by the create_user function

servicelist(self): returns the list of micro-services and websites where the user is either the author or the admin of the given account that he belongs to

accountservicelist(self): returns the list of micro-services and websites of the account of the logged user. security works as follows:
- The list returns all the micro-services and accounts where the user is NOT the author of the account whether the micro-service/website is private or not
- If the micro-service/website is 'forkable' then  the user cannot see the code and cannot fork

allservicelist(self): returns all micro-services/websites that are not part of the account of the logged user and are not private. Moreover, only those which are marked as forkable can be read (code) or forked.

connection(self,name): return a psycopg2 connection object when the micro-service has a database (POSTGres). The codeez package does not include the psycopg2 package for size and performance reasons, so it tries to import it and if it fails returns an error.
- If the microservice has a database but the psycopg2 is not installed, the function returns the host and port of the database, which always needs to be called with ssl mode as a requirement. The database name of the database is the name of the micro-service and an initial user is the user with the username and user authorization token.

connection = psycopg2.connect(host=host, port=port, user=self.User, password=self.UserAuthorization, dbname=name, sslmode='require')

logs(self,name): returns the logs of a micro-service/website as an array of all the containers running this micro-service:
["1":{logs of container 1},"2":{logs of container 2}]

create(self,name,directory,type='MService',Description=None,Fork=False,Private=False,DB=False,ServiceName=None,ServiceAccount=None):
To create a micro-service or a website:
name is the name of the micro-service/website. Only small letters are allowed.
directory is the directory where the code is located
type is either MService (Python) or Site (anything that can be served by nginx).
Desription
Fork: if you allow other users to forkread your micro-service/website. Requires Private to be set to False.
Private: Only you and your account admin can see/fork the micro-service/website
DB: creates a Postgre database in case DB is set to yes
ServiceName: in case of a 'Site', this is to attach a backend micro-service to the website. Calls to this backend micro-services are forward via /api/...
ServiceAccount: Account of the micro-service that serves as the backend for the website

This function returns the codeez url of the micro-service/website

update(self,name,directory,type='MService',Description=None,DB=False,Fork=False,Private=False,ServiceName=None,ServiceAccount=None):
Same as create. When updating, only the code is modified. The URL, host, etc. remain unchanged. If the microservice/website run on two containers, both containers are updated one after the other.
If there is not database at creation time, the update creates the database when the DB is set to true

delete((self,name):
To delete a micro-service/website. If a Database is associated, the database is deleted as well.

stop and start(self,name):
To stop/start a micro-service/website. This function is very useful to reduce cost in case the micro-service needs to be used only during certain hours or when the micro-servicewebsite can be deleted after a certain period of time

store(self,name,Content):
To store, independently of the runtime container a byte stream (content) and give a name to the storage (name). Uses the S3 standard
To retrieve the content of a given name

There are three possibilities to access storage for codeez.

1. By calling the store function in a codeez micro-service:

WARNING: when importing the store function in a codeez micro-service, it works slightly differently:

from storage import store
from storage import retrieve

a=store('name','content','User','UserKey')

2. By using the codeez package in any Python script (including a codeez micro-service)

import codeez

a=codeez.codeez('User','UserKey')

a.store('name','content')

3. By using the generic codeez API: https://storage.codeez.tech/myfunction/store?Name='+Name

where:
headers['User']='User'
headers['Authorization']='UserKey'

and body=content

Same to retrieve the content



