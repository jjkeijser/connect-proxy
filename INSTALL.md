# connect-proxy
Make socket connection using SOCKS4/5, telnet HTTP or HTTPS tunnel.

*************************************************************************

QUICK START:

  Unix:
		gcc -o connect connect.c -lssl -lcrypto

*************************************************************************

The development version can be found here:

   https://github.com/jjkeijser/connect-proxy/


How To Compile
==============
On Linux/UNIX environment:

		gcc -o connect connect.c -lssl -lcrypto

Or using a specific OpenSSL installation:

		gcc -o connect connect.c  -I../openssl-1.1.1g/include 
				-L../openssl-1.1.1g -lssl -lcrypto 

The default CA certificate file is the RHEL/CentOS/Fedora default:

		/etc/pki/tls/certs/ca-bundle.crt

You can specify an alternative location using

		gcc -o connect connect.c -D__DEFAULT_CA_PATH__=\"/some/path\"
				-lssl -lcrypto

(mind the quotes!)

