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
- download these modified files from GitHub to some folder, for example:
git clone https://github.com/CpServiceSpb/OpenSSLOcsp.git ocspmod
- replace original files at apps folder with modified ones;
- config with your options or without it;
- make;
- make test (if it is ncessary for you) ;
- make install.

