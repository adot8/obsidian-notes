
# Create Listeners

1.  Go to **Cobalt Strike > Listeners** to bring up the Listeners tab.
2.  Click **Add** to create a new listener.

Add the following listeners:

## HTTP

1.  Name: http
2.  Payload: Beacon HTTP
3.  HTTP Hosts: www.bleepincomputer.com
4.  HTTP Host (Stager) : www.bleepincomputer.com

## SMB

1.  Name: smb
2.  Payload: Beacon SMB
3.  Pipename: W32TIME_ALT-1337

## TCP

1.  Name: tcp
2.  Payload: Beacon TCP
3.  Port: 4444
4.  Bind to localhost: False

## TCP (local)

1.  Name: tcp-local
2.  Payload: Beacon TCP
3.  Port: 1337
4.  Bind to localhost: True