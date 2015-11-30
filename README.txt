# OpenSSLOcsp
Short description:
These files contains modified code for OpenSSl Ocsp acting as responder (aka server) 
using OpenSSL text index file as DB for storing Root certificate, responder key and responder certificate 
for each issued certificate at the index file.
It are intended all for OpenSSL 1.0.2d official released version only !
There are 3 modified files:
- apps.h
- ca.c
- ocsp.c
All are located at OSSL folder/apps folder.
And it is source code.

Building/installation:
These are fully coded files, not diffs, or patches.
That is to implement this code to OpenSSL source code, you should:
- download OpenSSL 1.0.2 released version from http://openssl.org/source/openssl-1.0.2d.tar.gz, for example
wget http://openssl.org/source/openssl-1.0.2d.tar.gz
- untar it, for example 
tar -xvzf openssl-1.0.2d.tar.gz
- download these modified files from GitHub to ocspmod folder, for example:
git clone https://github.com/CpServiceSpb/OpenSSLOcsp.git ocspmod
- replace original files at apps folder with modified ones;
- config with your options or without it;
- make;
- make test (if it is ncessary for you) ;
- make install.

Brief intro:
The main difference of this OSSL Ocsp version that now OCSP responder may uses OSSL txt database !

There are 3 new "fields" as new, added for each DB line that is for each Issued certificate: 
/path/Root certificate <Tab> /path/Ocsp responder certificate <Tab> /path/Ocsp responder key
for each issuing certificate additionally to existing ones.
Changed index text DB will be shown as following:
V	171125154741Z		202	unknown	/issuer of cert1 /path/RootCA.crt	/path/Resp1.crt	/path/Resp1.key
V	171125154742Z		203	unknown	/issuer of cert1 /path/RootCA.crt	/path/Resp2.crt	/path/Resp2.key

The 3 last fields are respectivelly: 
/path/Root certificate - the same as -CA parameter;
/path/Ocsp signer certificate - the same as -rsigner parameter;
/path/Ocsp signer key - the same as -rkey parameter.

Also 2 new command line switches "-ocspdb" & "-ocspdbsncert" are added usign in conjuction with "ocsp" .

Warning point:
As following within this mod, there are 2 Ocsp working modes:
- "command switcher" Ocsp responder mode - when "-CA" and/or "-rsigner" and/or "-rkey" switcher/s is/are used, 
of course with "-Index" without "-ocspdb" . 
In the case, OSSL will be started at Ocsp responder mode as at past 
using Root certificate and responder certificates/keys specified at mentioned parameters, 
that is Ocsp responder will get RootCA, responder key & certificate from specified files 
and will serve only request for one certificate. Let's call "current" mode.
- "index DB" Ocsp responder mode - when "-ocspdb" is used, of course, 
with "-Index" , without "-CA" and/or "-rsigner" and/or "-rkey" switchers. 
In the case, OSSL will be started at Ocsp responder mode using index DB file, 
that is Ocsp responder will get RootCA, responder key & certificate 
from index DB file for appropriate certificate 
and will serve many requests for as many certificates as presented at DB and 
which are Root certificates, responder certificates/keys presented for. Let's call "new" mode.

Determination of verifing certificate, if it exists or not at index DB, is carried out by serial number 
of requested certificate during getting Ocsp request as at "current" mode.
But then if necessary certificate exists at index DB, for making Ocsp respond (answer) , 
Root certiicate, responder certificate/key at 3 last fields from index DB are used.
There is not necessity to specify Root certiicate, responder certificate/key each time 
you launch OSSL at Ocsp responder mode.
Only once trinity of: Root certiicate, responder certificate/key 
for each issued certificate have to be filled at index DB 
and OSSL Ocsp responder can handle Ocsp request/respond 
for any certificate from index DB without reloading OSSL (if port is specified) .

Operation with index text DB:
First initialization:
By default, /path/file of cerficate which is Root for issuing one is added 
to index text DB to RootCA field during issuing certificate.
For other two parameters, that are for fields: responder key & certificate, 
"unknown" value is put at index text DB file.

Followng corrections:
Operation with index text DB:
To add/change "Root certificate" and/or "Ocsp signer certificate" and/or "Ocsp signer" key 
to OSSL text index DB fields, appropriate parameter or parameters should be specified 
with "-ocspdbscert" switch within ocsp also.
To delete, that is to clean one parameter up to all these parameters in OSSL text index DB, 
"-ocspdbscert" is inputed with non existing files or illegal s/n or with "" (double quotation marks 
without any symbol inside) at switches such as "-CA" and/or "-rsigner" and/or "-rkey" .
"Cleaning" such parameter (Root certificate, and/or responder certificate/key) means
setting up "unknown" value to respective index DB field.
In both cases, of course, serial of certificate or file of certificate in its own, 
which Root certificate, responder certificate/key is added/cleared for, have to be specified 
at "-ocspdbscert" switch also.

Index text DB with set/unset some of last fields looks like this:
V	171125154741Z		202	unknown	/issuer of cert1 unknown	/path/Resp1.crt	/path/Resp1.key
V	171125154742Z		203	unknown	/issuer of cert1 unknown	unknown	unknown


Examples:
launch OSSL as responder at "current" mode - as it was:
openssl ocsp -index /path/IndexDB.txt -rsigner /path/RSign1.crt -rkey /path/RSign1.key 
-CA /path/RootCA.crt -port 50000
or
openssl ocsp -index /psth/IndexDB.txt -rsigner /path/RSign1.crt -rkey /path/RSign1.key 
-CA /path/RootCA.crt -reqin /path/some.req

launch OSSL as responder at "new" mode - as from now (for this mode, workign with "-port" parmeter 
is more preferable (optimal) then with "-reqin" :
openssl ocsp -ocspdb -index /path/IndexDB.txt -port 50000
or
openssl ocsp -ocspdb -index /path/IndexDB.txt -reqin /path/some.req

Add the trinity of Root certificate, responder certificte/key to Index DB for certificate 
with serila number 203 from DB:
openssl ocsp -ocspdbsncert 203 -index /path/IndexDB.txt -rsigner -rsigner /path/RSign1.crt 
-rkey /path/RSign1.key -CA /path/RootCA.crt
or for certificate /path/server1.crt
openssl ocsp -ocspdbsncert /path/server1.crt -index /path/IndexDB.txt -rsigner /path/RSign1.crt 
-rkey /path/RSign1.key -CA /path/RootCA.crt

Change, for example of responder key for certificate with serial number 4590,
previously it was /path/RSign203.key at index DB:
openssl ocsp -ocspdbsncert 4590 -index /path/IndexDB.txt -rkey /path/RSignNew.key
or the same but for certificate of /path/server1.crt
openssl ocsp -ocspdbsncert /path/server1.crt -index /path/IndexDB.txt -rkey /path/RSignNew.key

Delete (clear) , for example, Root certtificate for certificate with serial number 10109,
openssl ocsp -ocspdbsncert 10109 -index /path/IndexDB.txt -CA /path/RootC#$%.crt 
(but file /path/RootC#$%.crt does not exist at /path)
or 
openssl ocsp -ocspdbsncert 10109 -index /path/IndexDB.txt -CA ""
or the same for certificate /path/server34.crt
openssl ocsp -ocspdbsncert /path/server34.crt -index /path/IndexDB.txt -CA /path/RootC#$%.crt 
(but file /path/RootC#$%.crt does not exist at /path)
or
openssl ocsp -ocspdbsncert /path/server34.crt -index /path/IndexDB.txt -CA ""

Note, that serial number as specified serial number or serial number extracted for specified 
certificate have to be presented at index DB, otherwise, error will be appeared.



Some tech details:
Function init_responder was taken by me from middle November of 2015 Git version 
due to 1.0.2 released version released port for a quiet long time (up to 45 seconds) 
and was not be able to reuse it (not reuse socket) .

Tested with Strongswan 5.3.3/5.3.5 with 2 certificate at server: exact server certificate and
its Intermediate one (from Root -> Inter -> Server, all issued of OSSL) at Ubuntu 14.04 LTS.

Some optimization of code may be required.

P. S. Also CGI (sh) script (tested at Ubuntu) is ready as Apache2 site config allows to accept Ocsp requests
from clients, feeds it to OSSL at Ocsp responder (don' t matter at "current" or "new" mode) , takes back returned responce (answer) and send it to clients, make requests logging during it. Fow Windows (2008), such script is ready partially.
Such script may be provided (by me) by e-mail requests.
