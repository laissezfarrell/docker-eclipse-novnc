docker-eclipse-novnc
=========================

What is this?
-------------
This project is an example of how a Docker container can be
run on headless servers to provide access to XWindow apps to anyone with
an HTML5 capable web browser. In this case, we are running the Eclipse IDE and
OpenBox, OpenOffice, Firefox, xpdf, and DOSbox in the container.

To embed the app in a web page, we use VNC for remote access to X via x11vnc and noVNC.
NoVNC uses web sockets and client-side javascript to provide a zero install VNC 
client run inside the user's web browser. Both Xvfb and Xorg/Xdummy are supported so
you can run the container on headless servers.

How to build 
------------
First cd to the noVNC directory and create a self.pem certificate 
for noVNC. This is necessary because we want to force secure connections 
(via https and wss) between the user's web browser and the container. 
See the "Encrypted noVNC Sessions" section below for details on how to
set up the site certificate.

After you have a cert in self.pem, build the container with the command
```
sudo docker build -t docker-eclipse-novnc .
```

Run using the default password from the Dockerfile build script:
```
sudo docker run -i -t -p 6080:6080 -h your.hostname.here docker-eclipse-novnc
```

Better yet, run and set the passwords for VNC and user via environment variables like this:

```
sudo docker run -i -t -p 6080:6080 -e UBUNTUPASS=supersecret -e VNCPASS=secret \
   -h your.hostname.here docker-eclipse-novnc
```
You need to specify the hostname to the container so that it matches the
site certificate that you configured noVNC with, or pedantic web browsers will
frighten users with scary warnings. 

TO access the app, point your web browser at
    https://your.hostname.here:6080/vnc.html
or
    https://your.hostname.here:6080/vnc_auto.html

You will be prompted for the vnc password which was set to 'foobar' in the
Dockerfile build. You'll probably want to change that and also change the 
hardcoded password ('badpassword') for the ubuntu account created 
in the build process by specifying passwords when you run the container.

Note that the user can skip the VNC password prompt if you redirect them to 

 https://your.host.here:6080/vnc.html?&encrypt=1&autoconnect=1&password=foobar


Encrypted noVNC Sessions
------------------------
To enable encrypted connections, you need to (at a minimum) create a 
noVNC self.pem certificate file as describe here: 
   https://github.com/kanaka/websockify/wiki/Encrypted-Connections

Even better, get your private key signed by a known certificate authority,
so that users are not confronted with frightening warnings about untrusted sites. 

Note that you may run into trouble if you include the entire CA signing 
chain if you use a CA such as Commodo (at least on Chrome and Safari) so I 
have been running with self.pem containner only the private key and the 
CA signed cert. But Firefox seems to want to see the entire signing chain for certs issued 
by Commodo, or something. So - some work is still needed to get Firefox
to behave, but Safari and Chrome seem to work.

PROTIP: make sure that the read permissions are set to only allow root to read the
self.pem file, since you probably don't want users to get access to the private key.

Xvfb vs XDummy/Xorg
-------------------
There are two approaches that can be taken to set up a virtual framebuffer for the VNC
server. The Dockerfile.xvfb build will create a container that uses the ancient and venerable
Xvfb approach, while the Dockerfile.xorg creates a container the sets up an Xdummy framebuffer
in Xorg. For the Xorg approach, we also need to set up the xorg.conf config file so you might 
want take a look at the settings there.

The reason for considering the Xorg approach over Xvfb is Xorg+Xdummy support the randr 
dynamic screen resizing functions so there are fewer warnings thrown by apps like firefox,
and someday we might get clever about resizing on the fly, or take advantage of GLX extnesions.
See https://www.xpra.org/trac/wiki/Xdummy for details.

You'll need to copy Dockerfile.xvfb or Dockerfile.xorg to Dockerfile to build the appropriate
version for your situation.

Misc Notes
----------
The file openbox-config/.config is used to put some reasonable default settings in place for 
the X environment when run inside a web browser. Reasonable means things like placing the dock 
at the top of the page so users don't have to scroll their web page to find it.

Extending
---------

To add scripts to run at startup, add them to this folder, with a ```.sh``` extension:

```
/etc/startup.aux/
```

To add supervisord configs, add them to this folder:
```
/etc/supervisor/conf.d/
```

Other things related to this project
------------------------------------
This container is meant to be run with and connected to a data volume container. For security reasons, I have not hosted that container's data on GitHub.