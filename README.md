# connect-proxy
Make socket connection using SOCKS4/5, telnet HTTP or HTTPS tunnel.

Based on connect.c from Shun-ichi GOTO <gotoh@taiyo.co.jp>
* Added HTTPS proxy support
* Made code gcc-9 and valgrind clean

How To Compile
==============
On Linux/UNIX environment:

		$ gcc connect.c -o connect -lssl -lcrypto

How To Use
==========
* You can specify proxy method in an environment variable or in a command line option.
* usage:

		./connect [-dnhstx45] [-p local-port][-R resolve] [-w timeout] 
		[-S [user@]socks-server[:port]]
		[-H [user@]proxy-server[:port]]
    	[-T proxy-server[:port]  [-c telnet-proxy-command]
		[-X [user@]proxy-server[:port]]
		[--help]
		[--socks-server          [user@]socks-server[:port]]
		[--http-proxy            [user@]proxy-server[:port]]
		[--telnet-proxy          proxy-server[:port]
		[--https-proxy           [user@]proxy-server[:port]]
		[--https-proxy-certname  name]
		[--https-user-cert       certfile.pem]
		[--https-user-key        keyfile.pem]
		[--no-check-certificate]
		host port
	
* "host" and "port" is for the target hostname and port-number to connect to.
* The '-H' or '--http-proxy' option specifies a hostname and port number of the http proxy server to 
  relay. If port is omitted, 80 is used. You can specify this value in the environment variable 
  HTTP_PROXY and pass the '-h' option to use it.
* The '-X' or '--https-proxy' option specifies a hostname and port number of the https proxy server to 
  relay. If port is omitted, 443 is used. You can specify this value in the environment variable 
  HTTPS_PROXY and pass the '-x' option to use it.
* The '-S' or '--socks-proxy' option specifies the hostname and port number of the SOCKS server to 
  relay.  Like '-H', port number can be omitted and the default is 1080. You can also specify this 
  value pair in the environment variable SOCKS5_SERVER and give the '-s' option to use it.
* The '-4' and the '-5' options are for specifying SOCKS relaying and  indicates protocol version 
  to use. It is valid only when used with '-s' or '-S'. Default is '-5' (protocol version 5)
* The '-R' option is for specifying method to resolve the hostname. Three keywords ("local", 
  "remote", "both") or dot-notation IP address are acceptable.  The keyword "both" means, "Try local 
  first, then remote". If a dot-notation IP address is specified, use this host as nameserver. The 
  default is "remote" for SOCKS5 or "local" for others. On SOCKS4 protocol, remote resolving method 
  ("remote" and "both") requires protocol 4a supported server.
* The '-p' option will forward a local TCP port instead of using the standard input and output.
* The '-P' option is same to '-p' except keep remote session. The program repeats waiting the port 
  with holding remote session without
  disconnecting. To disconnect the remote session, send EOF to stdin or kill the program.
* The '-w' option specifys timeout seconds for making connection with TARGET host.
* The '-d' option is used for debug. If you fail to connect, use this and check request to and 
  response from server.
 
 You can omit the "port" argument when program name is special format containing port number 
itself. For example,
 
		$ ln -s connect connect-25
 means this connect-25 command is spcifying port number 25 already so you need not 2nd argument 
(and ignored if specified).
* To use proxy, this example is for SOCKS5 connection to connect to 'host' at port 25 via SOCKS5 
server on 'firewall' host.

		$ connect -S firewall  host 25
  or
  
  		$ SOCKS5_SERVER=firewall; export SOCKS5_SERVER
		$ connect -s host 25
* For a HTTP-PROXY connection:

		$ connect -H proxy-server:8080  host 25
  or
  
		$ HTTP_PROXY=proxy-server:8080; export HTTP_PROXY
		$ connect -h host 25
* For a HTTPS-PROXY connection:

		$ connect -H proxy-server:443  host 25
  or

		$ HTTPS_PROXY=proxy-server:443; export HTTPS_PROXY
		$ connect -x host 25
  
TIPS
====
* Connect.c doesn't have any configuration to specify the SOCKS server.
  If you are a mobile user, this limitation might bother you.  However,
  You can compile connect.c and link with other standard SOCKS library
  like the NEC SOCKS5 library or Dante. This means connect.c is
  socksified and uses a configration file like to other SOCKSified
  network commands and you can switch configuration file any time
  (ex. when ppp startup) that brings you switching of SOCKS server for
  connect.c in same way with other commands. For this case, you can
  write ~/.ssh/config like this:
  
		ProxyCommand connect -n %h %p
 
SOCKS5 authentication
=====================
* Only USER/PASS authentication is supported.
 
HTTP Proxy authentication
=========================
* Only BASIC scheme is supported.

HTTPS proxy authentication
==========================
* BASIC scheme is supported.
* The server certificate can be verified against a CA certificate (or list of CA
  certficates) by specifying either '--https-ca-file' or '--https-ca-path'.
  (default file: /etc/pki/tls/certs/ca-bundle.crt).
* By default, the server certificate name (/CN=...) is checked against the hostname
  of the https_proxy server. It is possible to specify an alternative name using
  '--http-proxy-certname'.
* You can disable server certificate verification by specifying '--no-certificate-check'.
* Certificate based authentication is supported. Use the '--https-user-cert' and 
  '--https-user-key' parameters to specify the user certificate and key. If the private
  key is protected using a passphrase, the $SSH_ASKPASS program will be used to query the user.

The following environment variables can be used to specify the above parameters:
* HTTPS proxy server:		$HTTPS_PROXY
* proxy user:				$HTTPS_PROXY_USER
* proxy password:			$HTTPS_PROXY_PASSWORD
* server certificate name:	$HTTPS_PROXY_CERTNAME
* CA certificate name:		$HTTPS_PROXY_CA_FILE
* CA certificate path:		$HTTPS_PROXY_CA_PATH
* client certificate file:	$HTTPS_PROXY_USERCERT
* client privatekey file:	$HTTPS_PROXY_USERKEY

Authentication information
==========================
The User name for authentication is specifed by an environment variable or system login name.  And 
password is specified from environment variable or external program (specified in $SSH_ASKPASS) or 
tty.
The following environment variable is used for specifying user name.
- SOCKS: $SOCKS5_USER, $LOGNAME, $USER
- HTTP Proxy: $HTTP_PROXY_USER, $LOGNAME, $USER
- HTTPS Proxy: $HTTPS_PROXY_USER, $LOGNAME, $USER
 
ssh-askpass support
===================
You can use ssh-askpass (came from OpenSSH or else) to specify password on graphical environment 
(X-Window or MS Windows). To use this, set program name to environment variable SSH_ASKPASS. On 
UNIX, X-Window must be required, so $DISPLAY environment variable is also needed. On Win32 
environment, $DISPLAY is not mentioned.

