# OpenSSLOcsp
These files contains modified code for OpenSSl Ocsp acting as responder (aka server) using OpenSSL text index file as DB for storing Root certificate, responder key and responder certificate for each issued certificate at the index file.
It are intended all for OpenSSL 1.0.2d official released version only !
There are 3 modified files:
- apps.h
- ca.c
- ocsp.c
All are located at OSSL folder/apps folder.
And it is source code.

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

As following within this mod, there are 2 Ocsp working modes:
- "command switcher" Ocsp responder mode - when "-CA" and/or "-rsigner" and/or "-rkey" switcher/s is/are used, of course with "-Index" without "-ocspdb" . In the case, OSSL will be started at Ocsp responder mode as at past using Root certificate and responder certificates/keys specified at mentioned parameters, that is Ocsp responder will get RootCA, responder key & certificate from specified files and will serve only request for one certificate. Let's call "current" mode.
- "index DB" Ocsp responder mode - when "-ocspdb" is used, of course, with "-Index" , without "-CA" and/or "-rsigner" and/or "-rkey" switchers. In the case, OSSL will be started at Ocsp responder mode using index DB file, that is Ocsp responder will get RootCA, responder key & certificate from index DB file for appropriate certificate and will serve many requests for as many certificates as presented at DB and which are CAs, rsigner certificates & keys presented for. Let's call "new" mode.

By default, /path/file of cerficate which is Root for issuing one is added to index text DB to RootCA field during issuing certificate.
For other two parameters, that are for fields: responder key & certificate, "unknown" value is put at index text DB file.

To add/change "Root certificate" and/or "Ocsp signer certificate" and/or "Ocsp signer" key to OSSL text index DB fields, appropriate parameter or parameters should be specified with "-ocspdbscert" switch within ocsp also.
To delete, that is to clean one parameter up to all these parameters in OSSL text index DB, "-ocspdbscert" is inputed with non existing files or illegal s/n or with "" (double quotation marks without any symbol inside) at switches such as "-CA" and/or "-rsigner" and/or "-rkey" .
In both cases, of course, serial of certificate or file of certificate in its own, which Root certificate, respononder certificate/key is added/cleared for, have to be specified at "-ocspdbscert" switch also.

Function init_responder was taken by me from middle November of 2015 Git version due to 1.0.2 released version released port for a quiet long time (up to 45 seconds) and was not be able to reuse it.



