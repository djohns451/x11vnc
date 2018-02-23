#x11vnc: a VNC server for real X displays


   x11vnc allows one to view remotely and interact with real X displays
   (i.e. a display corresponding to a physical monitor, keyboard, and
   mouse) with any VNC viewer. In this way it plays the role for Unix/X11
   that WinVNC plays for Windows.

   It has built-in SSL/TLS encryption and 2048 bit RSA authentication,
   including VeNCrypt support; UNIX account and password login support;
   server-side scaling; single port HTTPS/HTTP+VNC; Zeroconf service
   advertising; and TightVNC and UltraVNC file-transfer. It has also been
   extended to work with non-X devices: natively on Mac OS X Aqua/Quartz,
   webcams and TV tuner capture devices, and embedded Linux systems such
   as Qtopia Core. Full IPv6 support is provided. More features are
   described here.

   It also provides an encrypted Terminal Services mode (-create, -svc,
   or -xdmsvc options) based on Unix usernames and Unix passwords where
   the user does not need to memorize his VNC display/port number.
   Normally a virtual X session (Xvfb) is created for each user, but it
   also works with X sessions on physical hardware. See the tsvnc
   terminal services mode of the SSVNC viewer for one way to take
   advantage of this mode.

   I wrote x11vnc back in 2002 because x0rfbserver was basically
   impossible to build on Solaris and had poor performance. The primary
   x0rfbserver build problems centered around esoteric C++ toolkits.
   x11vnc is written in plain C and needs only standard libraries and so
   should work on nearly all Unixes, even very old ones. I also created
   enhancements to improve the interactive response, added many features,
   and etc.

     _________________________________________________________________

    Background:

   VNC (Virtual Network Computing) is a very useful network graphics
   protocol (applications running on one computer but displaying their
   windows on another) in the spirit of X, however, unlike X, the
   viewing-end is very simple and maintains no state. It is a remote
   framebuffer (RFB) protocol.

   Some VNC links:
     * http://www.realvnc.com
     * http://www.tightvnc.com
     * http://www.ultravnc.com/
     * http://www.testplant.com/products/vine_server/OS_X

   For Unix, the traditional VNC implementation includes a "virtual" X11
   server Xvnc (usually launched via the vncserver command) that is not
   associated with a physical display, but provides a "fake" one X11
   clients (xterm, firefox, etc.) can attach to. A remote user then
   connects to Xvnc via the VNC client vncviewer from anywhere on the
   network to view and interact with the whole virtual X11 desktop.

   The VNC protocol is in most cases better suited for remote connections
   with low bandwidth and high latency than is the X11 protocol because
   it involves far fewer "roundtrips" (an exception is the cached pixmap
   data on the viewing-end provided by X.) Also, with no state maintained
   the viewing-end can crash, be rebooted, or relocated and the
   applications and desktop continue running. Not so with X11.

   So the standard Xvnc/vncserver program is very useful, I use it for
   things like:
     * Desktop conferencing with other users (e.g. code reviews.)
     * Long running apps/tasks I want to be able to view from many places
       (e.g. from home and work.)
     * Motif, GNOME, and similar applications that would yield very poor
       performance over a high latency link.

   However, sometimes one wants to connect to a real X11 display (i.e.
   one attached to a physical monitor, keyboard, and mouse: a Workstation
   or a SunRay session) from far away. Maybe you want to close down an
   application cleanly rather than using kill, or want to work a bit in
   an already running application, or would like to help a distant
   colleague solve a problem with their desktop, or would just like to
   work out on the deck for a while. This is where x11vnc is useful.
     _________________________________________________________________

    How to use x11vnc:

   In this basic example let's assume the remote machine with the X
   display you wish to view is "far-away.east:0" and the workstation you
   are presently working at is "sitting-here.west".

   Step 0. Download x11vnc (see below) and have it available to run on
   far-away.east (on some linux distros it is as easy as "apt-get install
   x11vnc", "emerge x11vnc", etc.) Similarly, have a VNC viewer (e.g.
   vncviewer) ready to run on sitting-here.west. We recommend TightVNC
   Viewers (see also our SSVNC viewer.)

   Step 1. By some means log in to far-away.east and get a command shell
   running there. You can use ssh, or even rlogin, telnet, or any other
   method to do this. We do this because the x11vnc process needs to be
   run on the same machine the X server process is running on (otherwise
   things would be extremely slow.)

   Step 2. In that far-away.east shell (with command prompt "far-away>"
   in this example) run x11vnc directed at the far-away.east X session
   display:

  far-away> x11vnc -display :0

   You could have also set the environment variable DISPLAY=:0 instead of
   using "-display :0". This step attaches x11vnc to the far-away.east:0
   X display (i.e. no viewer clients yet.)

   Common Gotcha: To get X11 permissions right, you may also need to set
   the XAUTHORITY environment variable (or use the -auth option) to point
   to the correct MIT-MAGIC-COOKIE file (e.g. /home/joe/.Xauthority.) If
   x11vnc does not have the authority to connect to the display it exits
   immediately. More on how to fix this below.

   If you suspect an X11 permissions problem do this simple test: while
   sitting at the physical X display open a terminal window
   (gnome-terminal, xterm, etc.) You should be able to run x11vnc
   successfully in that terminal without any need for command line
   options. If that works OK then you know X11 permissions are the only
   thing preventing it from working when you try to start x11vnc via a
   remote shell. Then fix this with the tips below.

   Note as of Feb/2007 you can also try the -find option instead of
   "-display ..." and see if that finds your display and Xauthority. Note
   as of Dec/2009 the -findauth and "-auth guess" options may be helpful
   as well.
   (End of Common Gotcha)

   When x11vnc starts up there will then be much chatter printed out (use
   "-q" to quiet it), until it finally says something like:
  .
  .
  13/05/2004 14:59:54 Autoprobing selected port 5900
  13/05/2004 14:59:54 screen setup finished.
  13/05/2004 14:59:54
  13/05/2004 14:59:54 The VNC desktop is far-away:0
  PORT=5900

   which means all is OK, and we are ready for the final step.

   Step 3. At the place where you are sitting (sitting-here.west in this
   example) you now want to run a VNC viewer program. There are VNC
   viewers for Unix, Windows, MacOS, Java-enabled web browsers, and even
   for PDA's like the Palm Pilot and Cell Phones! You can use any of them
   to connect to x11vnc (see the above VNC links under "Background:" on
   how to obtain a viewer for your platform or see this FAQ. For Solaris,
   vncviewer is available in the Companion CD package SFWvnc.)

   In this example we'll use the Unix vncviewer program on sitting-here
   by typing the following command in a second terminal window:

  sitting-here> vncviewer far-away.east:0

   That should pop up a viewer window on sitting-here.west showing and
   allowing interaction with the far-away.east:0  X11 desktop. Pretty
   nifty! When finished, exit the viewer: the remote x11vnc process will
   shutdown automatically (or you can use the -forever option to have it
   wait for additional viewer connections.)

   Common Gotcha: Nowadays there will likely be a host-level firewall on
   the x11vnc side that is blocking remote access to the VNC port (e.g.
   5900.) You will either have to open up that port (or a range of ports)
   in your firewall administration tool, or try the SSH tunnelling method
   below (even still the firewall must allow in the SSH port, 22.)


   Shortcut: Of course if you left x11vnc running on far-away.east:0 in a
   terminal window with the -forever option or as a service, you'd only
   have to do Step 3 as you moved around. Be sure to use a VNC Password
   or other measures if you do that.


   Super Shortcut: Here is a potentially very easy way to get all of it
   working.
     * Have x11vnc (0.9.3 or later) available to run on the remote host
       (i.e. in $PATH.)
     * Download and unpack a SSVNC bundle (1.0.19 or later, e.g.
       ssvnc_no_windows-1.0.28.tar.gz) on the Viewer-side machine.
     * Start the SSVNC Terminal Services mode GUI: ./ssvnc/bin/tsvnc
     * Enter your remote username@hostname (e.g. fred@far-away.east) in
       the "VNC Terminal Server" entry.
     * Click "Connect".

   That will do an SSH to username@hostname and start up x11vnc and then
   connect a VNC Viewer through the SSH encrypted tunnel.

   There are a number of things assumed here, first that you are able to
   SSH into the remote host; i.e. that you have a Unix account there and
   the SSH server is running. On Unix and MacOS X it is assumed that the
   ssh client command is available on the local machine (on Windows a
   plink binary is included in the SSVNC bundle.) Finally, it is assumed
   that you are already logged into an X session on the remote machine,
   e.g. your workstation (otherwise, a virtual X server, e.g. Xvfb, will
   be started for you.)

   In some cases the remote SSH server will not run commands with the
   same $PATH that you normally have in your shell there. In this case
   click on Options -> Advanced -> X11VNC Options, and type in the
   location of the x11vnc binary under "Full Path". (End of Super
   Shortcut)


   Desktop Sharing: The above more or less assumed nobody was sitting at
   the workstation display "far-away.east:0". This is often the case: a
   user wants to access her workstation remotely. Another usage pattern
   has the user sitting at "far-away.east:0" and invites one or more
   other people to view and interact with his desktop. Perhaps the user
   gives a demo or presentation this way (using the telephone for vocal
   communication.) A "Remote Help Desk" mode would be similar: a
   technician connects remotely to the user's desktop to interactively
   solve a problem the user is having.

   For these cases it should be obvious how it is done. The above steps
   will work, but more easily the user sitting at far-away.east:0 simply
   starts up x11vnc from a terminal window, after which the guests would
   start their VNC viewers. For this usage mode the "-connect
   host1,host2" option may be of use to automatically connect to the
   vncviewers in "-listen" mode on the list of hosts.
     _________________________________________________________________

    Tunnelling x11vnc via SSH:

   The above example had no security or privacy at all. When logging into
   remote machines (certainly when going over the internet) it is best to
   use ssh, or use a VPN (for a VPN, Virtual Private Network, the above
   example should be pretty safe.)

   For x11vnc one can tunnel the VNC protocol through an encrypted ssh
   channel. It would look something like running the following commands:
  sitting-here> ssh -t -L 5900:localhost:5900 far-away.east 'x11vnc -localhost
-display :0'

   (you will likely have to provide passwords/passphrases to login from
   sitting-here into your far-away.east Unix account; we assume you have
   a login account on far-away.east and it is running the SSH server)

   And then in another terminal window on sitting-here run the command:
  sitting-here> vncviewer -encodings "copyrect tight zrle hextile" localhost:0

   Note: The -encodings option is very important: vncviewer will often
   default to "raw" encoding if it thinks the connection is to the local
   machine, and so vncviewer gets tricked this way by the ssh
   redirection. "raw" encoding will be extremely slow over a networked
   link, so you need to force the issue with -encodings "copyrect tight
   ...". Nowadays, not all viewers use the -encodings option, try
   "-PreferredEncoding=ZRLE" (although the newer viewers seem to
   autodetect well when to use raw or not.)

   Note that "x11vnc -localhost ..." limits incoming vncviewer
   connections to only those from the same machine. This is very natural
   for ssh tunnelling (the redirection appears to come from the same
   machine.) Use of a VNC password is also strongly recommended.

   Note also the -t we used above (force allocate pseudoterminal), it
   actually seems to improve interactive typing response via VNC!

   You may want to add the -C option to ssh to enable compression. The
   VNC compression is not perfect, and so this may help a bit. However,
   over a fast LAN you probably don't want to enable SSH compression
   because it can slow things down. Try both and see which is faster.

   If your username is different on the remote machine use something
   like: "fred@far-away.east" in the above ssh command line.

   Some VNC viewers will do the ssh tunnelling for you automatically, the
   TightVNC Unix vncviewer does this when the "-via far-away.east" option
   is supplied to it (this requires x11vnc to be already running on
   far-away.east or having it started by inetd(8).) See the 3rd script
   example below for more info.

   SSVNC:  You may also want to look at the Enhanced TightVNC Viewer
   (ssvnc) bundles because they contain scripts and GUIs to automatically
   set up SSH tunnels (e.g. the GUI, "ssvnc", does it automatically and
   so does this command: "ssvnc_cmd -ssh user@far-away.east:0") and can
   even start up x11vnc as well.

   The Terminal Services mode of SSVNC is perhaps the easiest way to use
   x11vnc. You just need to have x11vnc available in $PATH on the remote
   side (and can SSH to the host), and then on the viewer-side you type
   something like:
  tsvnc fred@far-away.east

   everything else is done automatically for you. Normally this will
   start a virtual Terminal Services X session (RAM-only), but if you
   already have a real X session up on the physical hardware it will find
   that one for you.

   Gateways:  If the machine you SSH into is not the same machine with
   the X display you wish to view (e.g. your company provides incoming
   SSH access to a gateway machine), then you need to change the above
   to, e.g.: "-L 5900:OtherHost:5900":
  sitting-here> ssh -t -L 5900:OtherHost:5900 gateway.east

   Where gateway.east is the internet hostname (or IP) of the gateway
   machine (SSH server.) 'OtherHost' might be, e.g., freds-pc or
   192.168.2.33 (it is OK for these to be private hostnames or private IP
   addresses, the host in -L is relative to the remote server side.)

   Once logged in, you'll need to do a second login (ssh, rsh, etc.) to
   the workstation machine 'OtherHost' and then start up x11vnc on it (if
   it isn't already running.) (The "-connect gateway:59xx" option may be
   another alternative here with the viewer already in -listen mode.) For
   an automatic way to use a gateway and have all the network traffic
   encrypted (including inside the firewall) see Chaining SSH's.

   These gateway access modes also can be done automatically for you via
   the "Proxy/Gateway" setting in SSVNC (including the Chaining SSH's
   case, "Double Proxy".)

   Firewalls/Routers:

   A lot of people have inexpensive devices for home or office that act
   as a Firewall and Router to the machines inside on a private LAN. One
   can usually configure the Firewall/Router from inside the LAN via a
   web browser.

   Often having a Firewall/Router sitting between the vncviewer and
   x11vnc will make it impossible for the viewer to connect to x11vnc.

   One thing that can be done is to redirect a port on the
   Firewall/Router to, say, the SSH port (22) on an inside machine (how
   to do this depends on your particular Firewall/Router, often the
   router config URL is http://192.168.100.1 See www.portforward.com for
   more info.) This way you reach these computers from anywhere on the
   Internet and use x11vnc to view X sessions running on them.

   Suppose you configured the Firewall/Router to redirect these ports to
   two internal machines:
  Port 12300 -> 192.168.1.3, Port 22 (SSH)
  Port 12301 -> 192.168.1.4, Port 22 (SSH)

   (where 192.168.1.3 is "jills-pc" and 192.168.1.4 is "freds-pc".) Then
   the ssh's would look something like:
  sitting-here> ssh -t -p 12300 -L 5900:localhost:5900 jill@far-away.east 'x11v
nc -localhost -display :0'
  sitting-here> ssh -t -p 12301 -L 5900:localhost:5900 fred@far-away.east 'x11v
nc -localhost -display :0'

   Where far-away.east means the hostname (or IP) that the
   Router/Firewall is using (for home setups this is usually the IP
   gotten from your ISP via DHCP, the site http://www.whatismyip.com/ is
   a convenient way to determine what it is.)

   It is a good idea to add some obscurity to accessing your system via
   SSH by using some high random port (e.g. 12300 in the above example.)
   If you can't remember it, or are otherwise not worried about port
   scanners detecting the presence of your SSH server and there is just
   one internal PC involved you could map 22:
  Port 22 -> 192.168.1.3, Port 22 (SSH)

   Again, this SSH gateway access can be done automatically for you via
   the "Proxy/Gateway" setting in SSVNC. And under the "Remote SSH
   Command" setting you can enter the x11vnc -localhost -display :0.

   Host-Level-Firewalls: even with the hardware Firewall/Router problem
   solved via a port redirection, most PC systems have their own Host
   level "firewalls" enabled to protect users from themselves. I.e. the
   system itself blocks all incoming connections. So you will need to see
   what is needed to configure it to allow in the port (e.g. 22) that you
   desire. E.g. Yast, Firestarter, iptables(1), etc..

   VNC Ports and Firewalls: The above discussion was for configuring the
   Firewall/Router to let in port 22 (SSH), but the same thing can be
   done for the default VNC port 5900:
  Port 5900 -> 192.168.1.3, Port 5900 (VNC)
  Port 5901 -> 192.168.1.4, Port 5900 (VNC)

   (where 192.168.1.3 is "jills-pc" and 192.168.1.4 is "freds-pc".) This
   could be used for normal, unencrypted connections and also for SSL
   encrypted ones.

   The VNC displays to enter in the VNC viewer would be, say,
   "far-away.east:0" to reach jills-pc and "far-away.east:1" to reach
   freds-pc. We assume above that x11vnc is using port 5900 (and any
   Host-Level-firewalls on jills-pc has been configured to let that port
   in.) Use the "-rfbport" option to tell which port x11vnc should listen
   on.

   For a home system one likely does not have a hostname and would have
   to use the IP address, say, "24.56.78.93:0". E.g.:
  vncviewer 24.56.78.93:0

   You may want to choose a more obscure port on the router side, e.g.
   5944, to avoid a lot of port scans finding your VNC server. For 5944
   you would tell the viewer to use:
  vncviewer 24.56.78.93:44

   The IP address would need to be communicated to the person running the
   VNC Viewer. The site http://www.whatismyip.com/ can help here.

     _________________________________________________________________

   Scripts to automate ssh tunneling: As discussed below, there may be
   some problems with port 5900 being available. If that happens, the
   above port and display numbers may change a bit (e.g. -> 5901 and :1).
   However, if you "know" port 5900 will be free on the local and remote
   machines, you can easily automate the above two steps by using the
   x11vnc option -bg (forks into background after connection to the
   display is set up) or using the -f option of ssh. Some example scripts
   are shown below. Feel free to try the ssh -C to enable its compression
   and see if that speeds things up noticeably.
     _________________________________________________________________

   #1. A simple example script, assuming no problems with port 5900 being
   taken on the local or remote sides, looks like:
#!/bin/sh
# usage: x11vnc_ssh <host>:<xdisplay>
#  e.g.: x11vnc_ssh snoopy.peanuts.com:0
#  (user@host:N also works)

host=`echo $1 | awk -F: '{print $1}'`
disp=`echo $1 | awk -F: '{print $2}'`
if [ "x$disp" = "x" ]; then disp=0; fi

cmd="x11vnc -display :$disp -localhost -rfbauth .vnc/passwd"
enc="copyrect tight zrle hextile zlib corre rre raw"

ssh -f -t -L 5900:localhost:5900 $host "$cmd"

for i in 1 2 3
do
        sleep 2
        if vncviewer -encodings "$enc" :0; then break; fi
done

   See also rx11vnc.pl below.
     _________________________________________________________________

   #2. Another method is to start the VNC viewer in listen mode
   "vncviewer -listen" and have x11vnc initiate a reverse connection
   using the -connect option:
#!/bin/sh
# usage: x11vnc_ssh <host>:<xdisplay>
#  e.g.: x11vnc_ssh snoopy.peanuts.com:0
#  (user@host:N also works)

host=`echo $1 | awk -F: '{print $1}'`
disp=`echo $1 | awk -F: '{print $2}'`
if [ "x$disp" = "x" ]; then disp=0; fi

cmd="x11vnc -display :$disp -localhost -connect localhost"   # <== note new opt
ion
enc="copyrect tight zrle hextile zlib corre rre raw"

vncviewer -encodings "$enc" -listen &
pid=$!
ssh -t -R 5500:localhost:5500 $host "$cmd"
kill $pid

   Note the use of the ssh option "-R" instead of "-L" to set up a remote
   port redirection.
     _________________________________________________________________

   #3. A third way is specific to the TightVNC vncviewer special option
   -via for gateways. The only tricky part is we need to start up x11vnc
   and give it some time (5 seconds in this example) to start listening
   for connections (so we cannot use the TightVNC default setting for
   VNC_VIA_CMD):
#!/bin/sh
# usage: x11vnc_ssh <host>:<xdisplay>
#  e.g.: x11vnc_ssh snoopy.peanuts.com:0

host=`echo $1 | awk -F: '{print $1}'`
disp=`echo $1 | awk -F: '{print $2}'`
if [ "x$disp" = "x" ]; then disp=0; fi

VNC_VIA_CMD="ssh -f -t -L %L:%H:%R %G x11vnc -localhost -rfbport 5900 -display
:$disp; sleep 5"
export VNC_VIA_CMD

vncviewer -via $host localhost:0      # must be TightVNC vncviewer.

   Of course if you already have the x11vnc running waiting for
   connections (or have it started out of inetd(8)), you can simply use
   the TightVNC "vncviewer -via gateway host:port" in its default mode to
   provide secure ssh tunnelling.
     _________________________________________________________________



   VNC password file: Also note in the #1. example script that the option
   "-rfbauth .vnc/passwd" provides additional protection by requiring a
   VNC password for every VNC viewer that connects. The vncpasswd or
   storepasswd programs, or the x11vnc -storepasswd option can be used to
   create the password file. x11vnc also has the slightly less secure
   -passwdfile and "-passwd XXXXX" options to specify passwords.

   Very Important: It is up to YOU to tell x11vnc to use password
   protection (-rfbauth or -passwdfile), it will NOT do it for you
   automatically or force you to (use -usepw if you want to be forced
   to.) The same goes for encrypting the channel between the viewer and
   x11vnc: it is up to you to use ssh, stunnel, -ssl mode, a VPN, etc.
   (use the Enhanced TightVNC Viewer (SSVNC) GUI if you want to be forced
   to use SSL or SSH.) For additional safety, also look into the -allow
   and -localhost options and building x11vnc with tcp_wrappers support
   to limit host access.

     _________________________________________________________________

    Tunnelling x11vnc via SSL/TLS:

   One can also encrypt the VNC traffic using an SSL/TLS tunnel such as
   stunnel.mirt.net (also stunnel.org) or using the built-in (Mar/2006)
   -ssl openssl mode. A SSL-enabled Java applet VNC Viewer is also
   provided in the x11vnc package (and https can be used to download it.)

   Although not as ubiquitous as ssh, SSL tunnelling still provides a
   useful alternative. See this FAQ on -ssl and -stunnel modes for
   details and examples.

   The Enhanced TightVNC Viewer (SSVNC) bundles contain some convenient
   utilities to automatically set up an SSL tunnel from the viewer-side
   (i.e. to connect to "x11vnc -ssl ...".) And many other enhancements
   too.
     _________________________________________________________________

    Downloading x11vnc:

   x11vnc is a contributed program to the LibVNCServer project at
   SourceForge.net. I use libvncserver for all of the VNC aspects; I
   couldn't have done without it. The full source code may be found and
   downloaded (either file-release tarball or GIT tree) from the above
   link. As of Sep 2010, the x11vnc-0.9.12.tar.gz source package is
   released (recommended download). The x11vnc 0.9.12 release notes.

   The x11vnc package is the subset of the libvncserver package needed to
   build the x11vnc program. Also, you can get a copy of my latest,
   bleeding edge x11vnc-0.9.13-dev.tar.gz tarball to build the most up to
   date one.

   Precompiled Binaries/Packages:  See the FAQ below for information
   about where you might obtain a precompiled x11vnc binary from 3rd
   parties and some ones I create.

   VNC Viewers:  To obtain VNC viewers for the viewing side (Windows, Mac
   OS, or Unix) try these links:
     * http://www.tightvnc.com/download.html
     * http://www.realvnc.com/download-free.html
     * http://sourceforge.net/projects/cotvnc/
     * http://www.ultravnc.com/
     * Our Enhanced TightVNC Viewer (SSVNC)

       [ssvnc.gif]


   More tools: Here is a ssh/rsh wrapper script rx11vnc that attempts to
   automatically do the above Steps 1-3 for you (provided you have
   ssh/rsh login permission on the machine x11vnc is to be run on.) The
   above example would be: "rx11vnc far-away.east:0" typed into a shell
   on sitting-here.west. Also included is an experimental script
   rx11vnc.pl that attempts to tunnel the vnc traffic through an ssh port
   redirection (and does not assume port 5900 is free.) Have a look at
   them to see what they do and customize as needed:
     * rx11vnc wrapper script
     * rx11vnc.pl wrapper script to tunnel traffic thru ssh

     _________________________________________________________________

    Building x11vnc:

   Make sure you have all the needed build/compile/development packages
   installed (e.g. Linux distributions foolishly don't install them by
   default.) See this build FAQ for more details.

   If your OS has libjpeg.so and libz.so in standard locations you can
   build as follows (example given for the 0.9.12 release of x11vnc:
   replace with the version you downloaded):
(un-tar the x11vnc+libvncserver tarball)
# gzip -dc x11vnc-0.9.12.tar.gz | tar -xvf -

(cd to the source directory)
# cd x11vnc-0.9.12

(run configure and then run make)
# ./configure
# make

(if all went OK, copy x11vnc to the desired destination, e.g. $HOME/bin)
# cp ./x11vnc/x11vnc $HOME/bin

   Or do make install, it will probably install to /usr/local/bin (run
   ./configure --help for information on customizing your configuration,
   e.g. --prefix=/my/place.) You can now run it via typing "x11vnc",
   "x11vnc -help | more", "x11vnc -forever -shared -display :0", etc.


   Note: Currently gcc is recommended to build libvncserver. In some
   cases it will build with non-gcc compilers, but the resulting binary
   sometimes fails to run properly. For Solaris pre-built gcc binaries
   are at http://www.sunfreeware.com/. Some Solaris pre-built x11vnc
   binaries are here.

   However, one user reports it does work fine when built with Sun Studio
   10, so YMMV. In fact, here is a little build script to do this on
   Solaris 10:
#!/bin/sh
PATH=/usr/ccs/bin:/opt/SUNWspro/bin:$PATH; export PATH

CC='cc' \
CFLAGS='-xO4' \
LDFLAGS='-L/usr/sfw/lib -L/usr/X11/lib -R/usr/sfw/lib -R/usr/X11/lib' \
CPPFLAGS='-I /usr/sfw/include -I/usr/X11/include' \
./configure

MAKE="make -e"
AM_CFLAGS=""
export MAKE AM_CFLAGS
$MAKE

   In general you can use the "make -e" trick if you don't like
   libvncserver's choice of AM_CFLAGS. See the build scripts below for
   more ideas. Scripts similar to the above have been shown to work with
   vendor C compilers on HP-UX (ccom: HP92453-01) and Tru64 (Compaq C
   V6.5-011.)

   You can find information on Misc. Build problems here.

     _________________________________________________________________

   Building on Solaris, FreeBSD, etc:   Depending on your version of
   Solaris or other Unix OS the jpeg and/or zlib libraries may be in
   non-standard places (e.g. /usr/local, /usr/sfw, /opt/sfw, etc.)

   Note: If configure cannot find these two libraries then TightVNC and
   ZRLE encoding support will be disabled, and you don't want that!!! The
   TightVNC encoding gives very good compression and performance, it even
   makes a noticeable difference over a fast LAN.


   Shortcuts: On Solaris 10 you can pick up almost everything just by
   insuring that your PATH has /usr/sfw/bin (for gcc) and /usr/ccs/bin
   (for other build tools), e.g.:
  env PATH=/usr/sfw/bin:/usr/ccs/bin:$PATH sh -c './configure; make'

   (The only thing this misses is /usr/X11/lib/libXrandr.so.2, which is
   for the little used -xrandr option, see the script below to pick it up
   as well.)


   libjpeg is included in Solaris 9 and later (/usr/sfw/include and
   /usr/sfw/lib), and zlib in Solaris 8 and later (/usr/include and
   /usr/lib.) So on Solaris 9 you can pick up everything with something
   like this:
  env PATH=/usr/local/bin:/usr/ccs/bin:$PATH sh -c './configure --with-jpeg=/us
r/sfw; make'

   assuming your gcc is in /usr/local/bin and x11vnc 0.7.1 or later.
   These are getting pretty long, see those assignments split up in the
   build script below.


   If your system does not have these libraries at all you can get the
   source for the libraries to build them: libjpeg is available at
   ftp://ftp.uu.net/graphics/jpeg/ and zlib at http://www.gzip.org/zlib/.
   See also http://www.sunfreeware.com/ for Solaris binary packages of
   these libraries as well as for gcc. Normally they will install into
   /usr/local but you can install them anywhere with the
   --prefix=/path/to/anywhere, etc.


   Here is a build script that indicates one way to pass the library
   locations information to the libvncserver configuration via the
   CPPFLAGS and LDFLAGS environment variables.
---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8
<---
#!/bin/sh

# Build script for Solaris, etc, with gcc, libjpeg and libz in
# non-standard locations.

# set to get your gcc, etc:
#
PATH=/path/to/gcc/bin:/usr/ccs/bin:/usr/sfw/bin:$PATH

JPEG=/path/to/jpeg      # set to maybe "/usr/local", "/usr/sfw", or "/opt/sfw"
ZLIB=/path/to/zlib      # set to maybe "/usr/local", "/usr/sfw", or "/opt/sfw"

# Below we assume headers in $JPEG/include and $ZLIB/include and the
# shared libraries are in $JPEG/lib and $ZLIB/lib.  If your situation
# is different change the locations in the two lines below.
#
CPPFLAGS="-I $JPEG/include -I $ZLIB/include"
LDFLAGS="-L$JPEG/lib -R $JPEG/lib -L$ZLIB/lib -R $ZLIB/lib"

# These two lines may not be needed on more recent Solaris releases:
#
CPPFLAGS="$CPPFLAGS -I /usr/openwin/include"
LDFLAGS="$LDFLAGS -L/usr/openwin/lib -R /usr/openwin/lib"

# These are for libXrandr.so on Solaris 10:
#
CPPFLAGS="$CPPFLAGS -I /usr/X11/include"
LDFLAGS="$LDFLAGS -L/usr/X11/lib -R /usr/X11/lib"

# Everything needs to built with _REENTRANT for thread safe errno:
#
CPPFLAGS="$CPPFLAGS -D_REENTRANT"

export PATH CPPFLAGS LDFLAGS

./configure
make

ls -l ./x11vnc/x11vnc

---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8<---8
<---

   Then do make install or copy the x11vnc binary to your desired
   destination.

   BTW, To run a shell script, just cut-and-paste the above into a file,
   say "myscript", then modify the "/path/to/..." items to correspond to
   your system/environment, and then type: "sh myscript" to run it.

   Note that on Solaris make is /usr/ccs/bin/make, so that is why the
   above puts /usr/ccs/bin in PATH. Other important build utilities are
   there too: ld, ar, etc. Also, it is probably a bad idea to have
   /usr/ucb in your PATH while building.

   Starting with the 0.7.1 x11vnc release the "configure --with-jpeg=DIR
   --with-zlib=DIR" options are handy if you want to avoid making a
   script.

   If you need to link OpenSSL libssl.a on Solaris see this method.

   If you need to build on Solaris 2.5.1 or earlier or other older Unix
   OS's, see this workaround FAQ.


   Building on FreeBSD, OpenBSD, ...:   The jpeg libraries seem to be in
   /usr/local or /usr/pkg on these OS's. You won't need the openwin stuff
   in the above script (but you may need /usr/X11R6/....) Also starting
   with the 0.7.1 x11vnc release, this usually works:
  ./configure --with-jpeg=/usr/local
  make


   Building on HP-UX:   For jpeg and zlib you will need to do the same
   sort of thing as described above for Solaris. You set CPPFLAGS and
   LDFLAGS to find them (see below for an example.) You do not need to do
   any of the above /usr/openwin stuff. Also, HP-UX does not seem to
   support -R, so get rid of the -R items in LDFLAGS. Because of this, at
   runtime you may need to set LD_LIBRARY_PATH or SHLIB_PATH to indicate
   the directory paths so the libraries can be found. It is a good idea
   to have static archives, e.g. libz.a and libjpeg.a for the nonstandard
   libraries so that they get bolted into the x11vnc binary (and so won't
   get "lost".)

   Here is what we recently did to build x11vnc 0.7.2 on HP-UX 11.11
./configure --with-jpeg=$HOME/hpux/jpeg --with-zlib=$HOME/hpux/zlib
make

   Where we had static archives (libjpeg.a, libz.a) only and header files
   in the $HOME/hpux/... directories as discussed for the build script.

   On HP-UX 11.23 and 11.31 we have had problems compiling with gcc.
   "/usr/include/rpc/auth.h:87: error: field 'syncaddr' has incomplete
   type". As a workaround for x11vnc 0.9.4 and later set your CPPFLAGS to
   include:
  CPPFLAGS="-DIGNORE_GETSPNAM"
  export CPPFLAGS

   This disables a very rare usage mode for -unixpw_nis by not trying
   getspnam(3).

   Using HP-UX's C compiler on 11.23 and 11.31 we have some severe
   compiler errors that have not been worked around yet. If you need to
   do this, contact me and I will give you a drastic recipe that will
   produce a working binary.


   Building on AIX:   AIX: one user had to add the "X11.adt" package to
   AIX 4.3.3 and 5.2 to get build header files like XShm.h, etc. You may
   also want to make sure that /usr/lpp/X11/include, etc is being picked
   up by the configure and make.

   For a recent build on AIX 5.3 we needed to add these CFLAGS to be able
   to build with gcc:
  env CFLAGS='-maix64 -Xlinker -bbigtoc' ./configure ...

   we also built our own libjpeg and libz using -maix64.

   BTW, one way to run an Xvfb-like virtual X server for testing on AIX
   is something like "/usr/bin/X11/X -force -vfb -ac :1".


   Building on Mac OS X:   There is now native Mac OS X support for
   x11vnc by using the raw framebuffer feature. This mode does not use or
   need X11 at all. To build you may need to disable X11:
./configure --without-x ...
make

   However, if your system has the Mac OS X build package for X11 apps
   you will not need to supply the "--without-x" option (in this case the
   resulting x11vnc would be able to export both the native Mac OS X
   display and windows displayed in the XDarwin X server.) Be sure to
   include the ./configure option to find libjpeg on your system.


   OpenSSL:   Starting with version 0.8.3 x11vnc can now be built with
   SSL/TLS support. For this to be enabled the libssl.so library needs to
   be available at build time. So you may need to have additional
   CPPFLAGS and LDFLAGS items if your libssl.so is in a non-standard
   place. As of x11vnc 0.9.4 there is also the --with-ssl=DIR configure
   option.

   Note that from OpenSSL 1.1.0 on SSLv2 support has been dropped and
   SSLv3 deactivated at build time per default. This means that unless
   explicitly enabled, OpenSSL builds only support TLS (any version).
   Since there is a reason for dropping SSLv3 (heard of POODLE?), most
   distributions do not enable it for their OpenSSL binary. In summary
   this means compiling x11vnc against OpenSSL 1.1.0 or newer is no
   problem, but using encryption will require a viewer with TLS support.

   On Solaris using static archives libssl.a and libcrypto.a instead of
   .so shared libraries (e.g. from www.sunfreeware.com), we found we
   needed to also set LDFLAGS as follows to get the configure to work:
env LDFLAGS='-lsocket -ldl' ./configure --with-ssl=/path/to/openssl ...
make

     _________________________________________________________________

    Beta Testing:

   I don't have any formal beta-testers for the releases of x11vnc, so
   I'd appreciate any additional testing very much.

   Thanks to those who suggested features and helped beta test x11vnc
   0.9.12 released in Sep 2010!

   Please help test and debug the 0.9.13 version for release sometime in
   Winter 2010.

   The version 0.9.13 beta tarball is kept here:
   x11vnc-0.9.13-dev.tar.gz

   There are also some Linux, Solaris, Mac OS X, and other OS test
   binaries here. Please kick the tires and report bugs, performance
   regressions, undesired behavior, etc. to me.

   To aid testing of the built-in SSL/TLS support for x11vnc, a number of
   VNC Viewer packages for Unix, Mac OS X, and Windows have been created
   that provide SSL Support for the TightVNC Viewer (this is done by
   wrapper scripts and a GUI that starts STUNNEL.) It should be pretty
   convenient for automatic SSL and SSH connections. It is described in
   detail at and can be downloaded from the Enhanced TightVNC Viewer
   (SSVNC) page. The SSVNC Unix viewer also supports x11vnc's symmetric
   key encryption ciphers (see the 'UltraVNC DSM Encryption Plugin'
   settings panel.)


   Here are some features that will appear in the 0.9.13 release:
     * Improved support for non-X11 touchscreen devices (e.g. handheld or
       cell phone) via Linux uinput input injection. Additional tuning
       parameters are added. TSLIB touchscreen calibration is supported.
       Tested on Qtmoko Neo Freerunner. A tool, misc/uinput.pl, is
       provided to diagnose uinput behavior on new devices. The env.
       vars. X11VNC_UINPUT_BUS and X11VNC_UINPUT_VERSION are available if
       leaving them unset does not work.
     * The Linux uinput non-X11 input injection can now be bypassed:
       events can be directly written to the /dev/input/event devices
       specified by the user (direct_abs=..., etc.) A -pipeinput input
       injection helper script, misc/qt_tslib_inject.pl is provided as a
       tweakable non-builtin direct input injection method.
     * The list of new uinput parameters for the above two features is:
       pressure, tslib_cal, touch_always, dragskip, btn_touch;
       direct_rel, direct_abs, direct_btn, direct_key.
     * The MacOSX native server can now use OpenGL for the screen
       capture. In nearly all cases this is faster than the raw
       framebuffer capture method. There are build and run time flags,
       X11VNC_MACOSX_NO_DEPRECATED, etc. to disable use of deprecated
       input injection and screen access interfaces. Cursor shape now
       works for 64bit binaries.
     * The included SSL enabled Java VNC Viewers now handle Mouse Wheel
       events.
     * miscellaneous new features and changes:
     * In -reflect mode, the libvncclient connection can now have the
       pixel format modified via the environment variables
       X11VNC_REFLECT_bitsPerSample, X11VNC_REFLECT_samplesPerPixel, and
       X11VNC_REFLECT_bytesPerPixel
     * In -create mode the following environment variables are added to
       fine tune the behavior: FIND_DISPLAY_NO_LSOF: do not use lsof(1)
       to try to determine the Linux VT, FIND_DISPLAY_NO_VT_FIND: do not
       try to determine the Linux VT at all, X11VNC_CREATE_LC_ALL_C_OK:
       do not bother undoing the setting LC_ALL=C that the create_display
       script sets. The performance of the -create script has been
       improved for large installations (100's of user sessions on one
       machine.)
     * In -unixpw mode, one can now Tab from login: to Password.
     * An environment variable, X11VNC_SB_FACTOR, allows one to scale the
       -sb screenblank sleep time from the default 2 secs.
     * An experimental option -unixsock is available for testing. Note,
       however, that it requires a manual change to
       libvncserver/rfbserver.c for it to work.
     * Documented that -grabkbd is no longer working with some/most
       window managers (it can prevent resizing and menu posting.)


   Here are some features that appeared in the 0.9.12 release (Sep/2010):
     * One can now specify the maximum number of displays that can be
       created in -create mode via the env. var.
       X11VNC_CREATE_MAX_DISPLAYS
     * The X11VNC_NO_LIMIT_SHM env. var. is added to skip any automatic
       shared memory reduction.
     * The kdm display manager is now detected when trying not to get
       killed by the display manager.
     * A compile time bug is fixed so that configuring using
       --with-system-libvncserver pointing to LibVNCServer 0.9.7 works
       again. A bug from forced use of Xdefs.h is worked around.


   Here are some features that appeared in the 0.9.11 release (Aug/2010):
     * The source tree is synchronized with the most recent libvncclient
       (this only affects -reflect mode.) Build is fixed for
       incompatibilities when using an external LibVNCServer (e.g.
       ./configure --with-system-libvncserver...) Please help test these
       build and runtime aspects and report back what you find, thanks.
     * The SSL enabled Java VNC Viewer Makefile has been modified so that
       the jar files that are built are compatible back to Java 1.4.
     * In -create/-unixpw mode, the env. var. FD_USERPREFS may be set to
       a filename in the user's home directory that includes default
       username:options values (so the options do not need to be typed
       every time at the login prompt.)
     * In -reflect mode cursor position updates are now handled
       correctly.


   Here are some features that appeared in the 0.9.10 release (May/2010):
     * The included SSL enabled Java applet viewer now supports Chained
       SSL Certificates. The debugCerts=yes applet parameter aids
       troubleshooting certificate validation. The x11vnc -ssl mode has
       always supported chained SSL certificates (simply put the
       intermediate certificates, in order, after the server certificate
       in the pem file.)
     * A demo CGI script desktop.cgi shows how to create an SSL
       encrypted, multi-user x11vnc web login desktop service. The script
       requires x11vnc version 0.9.10. The user logs into a secure web
       site and gets his/her own virtual desktop (Xvfb.) x11vnc's SSL
       enabled Java Viewer Applet is launched by the web browser for
       secure viewing (and so no software needs to be installed on the
       viewer-side.) One can use the desktop.cgi script for ideas to
       create their own fancier or customized web login desktop service
       (e.g. user-creation, PHP, SQL, specialized desktop application,
       etc.) More info here. There is also an optional 'port redirection'
       mode that allows redirection to other SSL enabled VNC servers
       running inside the firewall.
     * Built-in support for IPv6 (128 bit internet addresses) is now
       provided. See the -6 and -connect options for details.
       Additionally, in case there are still problems with built-in IPv6
       support, a transitional tool is provided in inet6to4 that allows
       x11vnc (or any other IPv4 application) to receive connections over
       IPv6.
     * The Xdummy wrapper script for Xorg's dummy driver is updated and
       no longer requires being run as root. New service options are
       provided to select Xdummy over Xvfb as the virtual X server to be
       created.
     * The "%" unix password verification tricks for the -unixpw option
       are now documented. They have also been extended to run a command
       as the user if one sets the environment variable UNIXPW_CMD. The
       desktop.cgi demo script takes advantage of this new feature.
     * A bug has been fixed that would prevent the Java applet viewer
       from being downloaded successfully in single-port HTTPS/VNC inetd
       mode. The env. var. X11VNC_HTTPS_DOWNLOAD_WAIT_TIME can be used to
       adjust for how many seconds a -inetd or -https httpd download is
       waited for (default 15 seconds.) The applet will now autodetect
       x11vnc and use GET=1 for faster connecting. Many other
       improvements and fixes.
     * The TightVNC security type (TightVNC features enabler) now works
       for RFB version 3.8.
     * The X property X11VNC_TRAP_XRANDR can be set on a desktop to force
       x11vnc to use the -xrandr screen size change trapping code.
     * New remote control query options: pointer_x, pointer_y,
       pointer_same, pointer_root, and pointer_mask. A demo script using
       them misc/panner.pl is provided.
     * The -sslScripts option prints out the SSL certificate management
       scripts.


   Here are some features that appeared in the 0.9.9 release (Dec/2009):
     * The -unixpw_system_greeter option, when used in combined unixpw
       and XDMCP FINDCREATEDISPLAY mode (for example: -xdmsvc), enables
       the user to press Escape to jump directly to the XDM/GDM/KDM login
       greeter screen. This way the user avoids entering his unix
       password twice at X session creation time. Also, the unixpw login
       panel now has a short help displayed if the user presses 'F1'.
     * x11vnc now tries to be a little bit more aggressive in keeping up
       with VNC client's framebuffer update requests. Some broken VNC
       clients like Eggplant and JollysFastVNC continuously spray these
       requests at VNC servers (regardless of whether they have received
       any updates or not.) Under some circumstances this could lead to
       x11vnc falling behind. The -extra_fbur option allows one to fine
       tune the setting. Additionally, one may also dial down delays:
       e.g. "-defer 5" and "-wait 5" (or to 1 or even 0) or -nonap or
       -allinput to keep up with these VNC clients at the expense of
       increased system load.
     * Heuristics are applied to try to determine if the X display is
       currently in a Display Manager Greeter Login panel (e.g. GDM) If
       so, x11vnc's creation of any windows and use of XFIXES are
       delayed. This is to try to avoid x11vnc being killed after the
       user logs in if the GDM KillInitClients=true is in effect. So one
       does not need to set KillInitClients=false. Note that in recent
       GDM the KillInitClients option has been removed. Also delayed is
       the use of the XFIXES cursor fetching functionality; this avoids
       an Xorg bug that causes Xorg to crash right after the user logs
       in.
     * A new option -findauth runs the FINDDISPLAY script that applies
       heuristics that try to determine the XAUTHORITY file. The use of
       '-auth guess' will use the XAUTHORITY that -findauth reveals. This
       can be handy in with the lastest GDM where the ability to store
       cookies in ~/.Xauthority has been removed. If x11vnc is running as
       root (e.g. inetd) and you add -env FD_XDM=1 to the above -findauth
       or -auth guess command lines, it will find the correct XAUTHORITY
       for the given display (this works for XDM/GDM/KDM if the login
       greeter panel is up or if someone has already logged into an X
       session.)
     * The FINDDISPLAY and FINDCREATEDISPLAY modes (i.e. "-display
       WAIT:cmd=...", -find, -create) now work correctly for the
       user-supplied login program scheme "-unixpw_cmd ...", as long as
       the login program supports running commands specified in the
       environment variable "RFB_UNIXPW_CMD_RUN" as the logged-in user.
       The mode "-unixpw_nis ..." has also been made more consistent.
     * The -stunnel option (like -ssl but uses stunnel as an external
       helper program) now works with the -ssl "SAVE" and "TMP" special
       certificate names. The -sslverify and -sslCRL options now work
       correctly in -stunnel mode. Single port HTTPS connections are also
       supported for this mode.
     * There is an experimental Application Sharing mode that improves
       upon the -id/-sid single window sharing: -appshare (run "x11vnc
       -appshare -help" for more info.) It is still very primitive and
       approximate, but at least it displays multiple top-level windows.
     * The remote control command -R can be used to instruct x11vnc to
       resend its most recent copy of the Clipboard, Primary, or
       Cutbuffer selections: "x11vnc -R resend_clipboard", "x11vnc -R
       resend_primary", and "x11vnc -R resend_cutbuffer".
     * The fonts in the GUI (-gui) can now by set via environment
       variables, e.g. -env X11VNC_FONT_BOLD='Helvetica -16 bold' and
       -env X11VNC_FONT_FIXED='Courier -14'.
     * The XDAMAGE mechanism is now automatically disabled for a period
       of time if a game or screensaver generates too many XDAMAGE
       rectangles per second. This avoids the X11 event queue from
       soaking up too much memory.
     * There is an experimental workaround: "-env X11VNC_WATCH_DX_DY=1"
       that tries to avoid problems with poorly constructed menu themes
       that place the initial position of the mouse cursor inside a menu
       item's active zone. More information can be found here.


   Here are some features that appeared in the 0.9.8 release (Jul/2009):
     * Stability improvements to -threads mode. Running x11vnc this way
       is more reliable now. Threaded operation sometimes gives better
       interactive response and faster updates: try it out. The threaded
       mode now supports multiple VNC viewers using the same VNC
       encoding. The threaded mode can also yield a performance
       enhancement in the many client case (e.g. class-room broadcast.)
       We have tested with 30 to 50 simultaneous clients. See also
       -reflect.
       For simultaneous clients: the ZRLE encoding is thread safe on all
       platforms, and the Tight and Zlib encodings are currently only
       thread safe on Linux where thread local storage, __thread, is
       used. If your non-Linux system and compiler support __thread one
       can supply -DTLS=__thread to enable it. When there is only one
       connected client, all encodings are safe on all platforms. Note
       that some features (e.g. scroll detection and -ncache) may be
       disabled or run with reduced functionality in -threads mode.
     * Automatically tries to work around an Xorg server and GNOME bug
       involving infinitely repeating keys when turning off key
       repeating. Use -repeat if the automatic workaround fails.
     * Improved reliability of the Single Port SSL VNC and HTTPS java
       viewer applet delivery mechanism.
     * The -clip mode works under -rawfb.


   Here are some features that appeared in the 0.9.7 release (Mar/2009):
     * Support for polling Linux Virtual Terminals (also called virtual
       consoles) directly instead of using /dev/fb. The option to use is,
       for example, "-rawfb vt2" for Virtual Terminal 2, etc. In this
       case the special file /dev/vcsa2 is used to retrieve vt2's current
       text. Text and colors are shown, but no graphics.
     * Support for less than 8 bits per pixel framebuffers (e.g. 4 or 1
       bpp) in the -rawfb mode.
     * The SSL enabled UltraVNC Java viewer applet now has a [Home] entry
       in the "drives" drop down menu. This menu can be configured with
       the ftpDropDown applet parameter. All of the applet parameters are
       documented in classes/ssl/README.
     * Experimental support for VirtualGL's TurboVNC (an enhanced
       TightVNC for fast LAN high framerate usage.)
     * The CUPS Terminal Services helper mode has been improved.
     * Improvements to the -ncache_cr that allows smooth opaque window
       motions using the 'copyrect' encoding when using -ncache mode.
     * The -rmflag option enables a way to indicate to other processes
       x11vnc has exited.
     * Reverse connections using anonymous Diffie Hellman SSL encryption
       now work.


   Here are some features that appeared in the 0.9.6 release (Dec/2008):
     * Support for VeNCrypt SSL/TLS encrypted connections. It is enabled
       by default in the -ssl mode. VNC Viewers like vinagre,
       gvncviewer/gtk-vnc, the vencrypt package, SSVNC, and others
       support this encryption mode. It can also be used with the -unixpw
       option to enable Unix username and password authentication
       (VeNCrypt's "*Plain" modes.) A similar but older VNC security type
       "ANONTLS" (used by vino) is supported as well. See the -vencrypt
       and -anontls options for additional control. The difference
       between x11vnc's normal -ssl mode and VeNCrypt is that the former
       wraps the entire VNC connection in SSL (like HTTPS does for HTTP,
       i.e. "vncs://") while VeNCrypt switches on the SSL/TLS at a
       certain point during the VNC handshake. Use -sslonly to disable
       both VeNCrypt and ANONTLS (vino.)
     * The "-ssl ANON" option enables Anonymous Diffie-Hellman (ADH) key
       exchange for x11vnc's normal SSL/TLS operation. Note that
       Anonymous Diffie-Hellman uses encryption for privacy, but provides
       no authentication and so is susceptible to Man-In-The-Middle
       attacks (and so we do not recommend it: we prefer you use "-ssl
       SAVE", etc. and have the VNC viewer verify the cert.) The ANONTLS
       mode (vino) only supports ADH. VeNCrypt mode supports both ADH and
       regular X509 SSL certificates modes. For these ADH is enabled by
       default. See -vencrypt and -anontls for how to disable ADH.
     * For x11vnc's SSL/TLS modes, one can now specify a Certificate
       Revocation List (CRL) with the -sslCRL option. This will only be
       useful for wide deployments: say a company-wide x11vnc SSL access
       deployment using a central Certificate Authority (CA) via
       -sslGenCA and -sslGenCert. This way if a user has his laptop lost
       or stolen, you only have to revoke his key instead of creating a
       new Certificate Authority and redeploying new keys to all users.
     * The default SSL/TLS mode, "-ssl" (no pem file parameter supplied),
       is now the same as "-ssl SAVE" and will save the generated
       self-signed cert in "~/.vnc/certs/server.pem". Previously "-ssl"
       would create a temporary self-signed cert that was discarded when
       x11vnc exited. The reason for the change is to at least give the
       chance for the VNC Viewer side (e.g. SSVNC) to remember the cert
       to authenticate subsequent connections to the same x11vnc server.
       Use "-ssl TMP" to regain the previous behavior. Use "-ssl
       SAVE_NOPROMPT" to avoid being prompted about using passphrase when
       the certificate is created.
     * The option -http_oneport enables single-port HTTP connections via
       the Java VNC Viewer. So, for example, the web browser URL
       "http://myhost.org:5900" works the same as
       "http://myhost.org:5800", but with the convenience of only
       involving one port instead of two. This works for both unencrypted
       connections and for SSH tunnels (see -httpsredir if the tunnel
       port differs.) Note that HTTPS single-port operation in -ssl SSL
       encrypted mode has been available since x11vnc version 0.8.3.
     * For the -avahi/-zeroconf Service Advertizing mode, if x11vnc was
       not compiled with the avahi-client library, then an external
       helper program, either avahi-publish(1) (on Unix) or dns-sd(1) (on
       Mac OS X), is used instead.
     * The "-rfbport PROMPT" option will prompt the user via the GUI to
       select the VNC port (e.g. 5901) to listen on, and a few other
       basic settings. This enables a handy GUI mode for naive users:
   x11vnc -gui tray=setpass -rfbport PROMPT -logfile $HOME/.x11vnc.log.%VNCDISP
LAY
       suitable for putting in a launcher or menu, e.g. x11vnc.desktop.
       The -logfile expansion is new too. In the GUI, the tray=setpass
       Properties panel has been improved.
     * The -solid solid background color option now works for the Mac OS
       X console.
     * The -reopen option instructs x11vnc to try to reopen the X display
       if it is prematurely closed by, say, the display manager (e.g.
       GDM.)


   Here are some features that appeared in the 0.9.5 release (Oct/2008):
     * Symmetric key encryption ciphers. ARC4, AES-128, AES-256,
       blowfish, and 3des are supported. Salt and initialization vector
       seeding is provided. These compliment the more widely used SSL and
       SSH encryption access methods. SSVNC also supports these
       encryption modes.
     * Scaling differently along the X- and Y-directions. E.g. "-scale
       1280x1024" or "-scale 0.8x0.75"    Also, "-geometry WxH" is an
       alias for "-scale WxH"
     * By having SSVNC version 1.0.21 or later available in your $PATH,
       the -chatwindow option allows a UltraVNC Text Chat window to
       appear on the local X11 console/display (this way the remote
       viewer can chat with the person at the physical display; e.g.
       helpdesk mode.) This also works on the Mac OS X console if the
       Xquartz X11 server (enabled by default on leopard) is running for
       the chatwindow.
     * The HTTP Java viewer applet jar, classes/VncViewer.jar, has been
       updated with an improved implementation based on the code used by
       the classes/ssl applets.


   Here are some features that appeared in the 0.9.4 release (Sep/2008):
     * Improvements to the -find and -create X session finding or
       creating modes: new desktop types and service redirection options.
       Personal cupsd daemon and SSH port redirection helper for use with
       SSVNC's Terminal Services feature.
     * Reverse VNC connections via -connect work in the -find, -create
       and related -display WAIT:... modes.
     * Reverse VNC connections (either normal or SSL) can use a Web Proxy
       or a SOCKS proxy, or a SSH connection, or even a CGI URL to make
       the outgoing connection. See: -proxy. Forward connections can also
       use: -ssh.
     * Reverse VNC connections via the UltraVNC repeater proxy (either
       normal or SSL) are supported. Use either the "-connect
       repeater=ID:NNNN+host:port" or "-connect
       repeater://host:port+ID:NNNN" notation. The SSVNC VNC viewer also
       supports the UltraVNC repeater. Also, a perl repeater implemention
       is here: ultravnc_repeater.pl
     * Support for indexed colormaps (PseudoColor) with depths other than
       8 (from 1 to 16 now work) for non-standard hardware. Option
       "-advertise_truecolor" to handle some workaround in this mode.
     * Support for the ZYWRLE encoding, this is the RealVNC ZRLE encoding
       extended to do motion video and photo regions more efficiently by
       way of a Wavelet based transformation.
     * The -finddpy and -listdpy utilities help to debug and configure
       the -find, -create, and -display WAIT:... modes.
     * Some automatic detection of screen resizes are handled even if the
       -xrandr option is not supplied.
     * The -autoport options gives more control over the VNC port x11vnc
       chooses.
     * The -ping secs can be used to help keep idle connections alive.
     * Pasting of the selection/clipboard into remote applications (e.g.
       Java) has been improved.
     * Fixed a bug if a client disconnects during the 'speed-estimation'
       phase.
     * To unset Caps_Lock, Num_Lock and raise all keys in the X server
       use -clear_all.
     * Usage with dvorak keyboards has been improved. See also: -xkb.
     * The Java Viewer applet source code is now included in the
       x11vnc-0.9.*.tar.gz tarball. This means you can now build the Java
       viewer applet jar files from source. If you stopped shipping the
       Java viewer applet jar files due to lack of source code, you can
       start again.


   Here are some features that appeared in the 0.9.3 release (Oct/2007):
     * Viewer-side pixmap caching. A large area of pixels (at least 2-3
       times as big as the framebuffer itself; the bigger the better...
       default is 10X) is placed below the framebuffer to act as a
       buffer/cache area for pixel data. The VNC CopyRect encoding is
       used to move it around, so any viewer can take advantage of it.
       Until we start modifying viewers you will be able to see the cache
       area if you scroll down (this makes it easier to debug!) For
       testing the default is "-ncache 10". The unix Enhanced TightVNC
       Viewer ssvnc has a nice -ycrop option to help hide the pixel cache
       area from view.


   Here are some features that appeared in the 0.9.2 release (Jun/2007):
     * Building with no OpenSSL libssl available (or with --without-ssl)
       has been fixed.
     * One can configure x11vnc via "./configure
       --with-system-libvncserver" to use a system installed libvncserver
       library instead of the one bundled in the release tarball.
     * If UltraVNC file transfer or chat is detected, then VNC clients
       are "pinged" more often to prevent these side channels from
       becoming serviced too infrequently.
     * In -unixpw mode in the username and password dialog no text will
       be echoed if the first character sent is "Escape". This enables a
       convenience feature in SSVNC to send the username and password
       automatically.


   Here are some features that appeared in the 0.9.1 release (May/2007):
     * The UltraVNC Java viewer has been enhanced to support SSL (as the
       TightVNC viewer had been previously.) The UltraVNC Java supports
       ultravnc filetransfer, and so can be used as a VNC viewer on Unix
       that supports ultravnc filetransfer. It is in the
       classes/ssl/UltraViewerSSL.jar file (that is pointed to by
       ultra.vnc.) The signed applet SignedUltraViewerSSL.jar version
       (pointed to by ultrasigned.vnc) will be needed to access the local
       drive if you are using it for file transfer via a Web browser.
       Some other bugs in the UltraVNC Java viewer were fixed and a few
       improvements to the UI made.
     * A new Unix username login mode for VNC Viewers authenticated via a
       Client SSL Certificate: "-users sslpeer=". The emailAddress
       subject field is inspected for username@hostname and then acts as
       though "-users +username" has been supplied. This way the Unix
       username is identified by (i.e. simply extracted from) the Client
       SSL Certificate. This could be useful with -find, -create and -svc
       modes if you are also have set up and use VNC Client SSL
       Certificate authentication.
     * For external display finding/creating programs (e.g. WAIT:cmd=...)
       if the VNC Viewer is authenticated via a Client SSL Certificate,
       then that Certificate is available in the environment variable
       RFB_SSL_CLIENT_CERT.


   Here are some features that appeared in the 0.9 release (Apr/2007):
     * VNC Service advertising via mDNS / ZeroConf / BonJour with the
       Avahi client library. Enable via "-avahi" or "-zeroconf".
     * Implementations of UltraVNC's TextChat, SingleWindow, and
       ServerInput extensions (requires ultravnc viewer or ssvnc Unix
       viewer.) They toggle the selection of a single window (-id), and
       disable (friendly) user input and viewing (monitor blank) at the
       VNC server.
     * Short aliases "-find", "-create", "-svc", and "-xdmsvc" for
       commonly used FINDCREATEDISPLAY usage modes.
     * Reverse VNC connections (viewer listening) now work in SSL (-ssl)
       mode.
     * New options to control the Monitor power state and keyboard/mouse
       grabbing: -forcedpms, -clientdpms, -noserverdpms, and -grabalways.
     * A simple way to emulate inetd(8) to some degree via the "-loopbg"
       option.
     * Monitor the accuracy of XDAMAGE and apply "-noxdamage" if it is
       not working well. OpenGL applications like like beryl and MythTv
       have been shown to make XDAMAGE not work properly.
     * For Java SSL connections involving a router/firewall port
       redirection, an option -httpsredir to spare the user from needing
       to include &PORT=NNN in the browser URL.


   Here are some features that appeared in the 0.8.4 release (Feb/2007):
     * Native Mac OS X Aqua/Quartz support. (i.e. OSXvnc alternative;
       some activities are faster)
     * A new login mode: "-display WAIT:cmd=FINDCREATEDISPLAY -unixpw
       ..." that will Create a new X session (either virtual or real and
       with or without a display manager, e.g. kdm) for the user if it
       cannot find the user's X session display via the FINDDISPLAY
       method. See the -svc and the -xdmsvc aliases.
     * x11vnc can act as a VNC reflector/repeater using the "-reflect
       host:N" option. Instead of polling an X display, the remote VNC
       Server host:N is connected to and re-exported via VNC. This is
       intended for use in broadcasting a display to many (e.g. > 16;
       classroom or large demo) VNC viewers where bandwidth and other
       resources are conserved by spreading the load over a number of
       repeaters.
     * Wireframe copyrect detection for local user activity (e.g. someone
       sitting at the physical display moving windows) Use
       -nowireframelocal to disable.
     * The "-N" option couples the VNC Display number to the X Display
       number. E.g. if your X DISPLAY is :2 then the VNC display will be
       :2 (i.e. using port 5902.) If that port is taken x11vnc will exit.
     * Option -nodpms to avoid problems with programs like KDE's
       kdesktop_lock that keep restarting the screen saver every few
       seconds.
     * To automatically fix the common mouse motion problem on XINERAMA
       (multi-headed) displays, the -xwarppointer option is enabled by
       default when XINERAMA is active.

   If you have a Mac please try out the native Mac OS X support, build
   with "./configure --without-x", or download a binary mentioned above,
   (even if you don't plan on ever using it in this mode!), and let me
   know how it went. Thanks.


   Here are some features that appeared in the 0.8.3 release (Nov/2006):
     * The -ssl option provides SSL encryption and authentication
       natively via the www.openssl.org library. One can use from a
       simple self-signed certificate server certificate up to full CA
       and client certificate authentication schemes.
     * Similar to -ssl, the -stunnel option starts up a SSL tunnel server
       stunnel (that must be installed separately on the system:
       stunnel.mirt.net ) to allow only encrypted SSL connections from
       the network.
     * The -sslverify option allows for authenticating VNC clients via
       their certificates in either -ssl or -stunnel modes.
     * Certificate creation and management tools are provide in the
       -sslGenCert, -sslGenCA, and related options.
     * An SSL enabled Java applet VNC Viewer applet is provided by x11vnc
       in classes/ssl/VncViewer.jar. In addition to normal HTTP, the
       applet may be loaded into the web browser via HTTPS (HTTP over
       SSL.) (one can use the VNC port, e.g. https://host:5900/, or also
       the separate -https port option.) A wrapper shell script
       ss_vncviewer is also provided that sets up a stunnel client-side
       tunnel on Unix systems. See Enhanced TightVNC Viewer (SSVNC) for
       other SSL/SSH viewer possibilities.
     * The -unixpw option supports Unix username and password
       authentication (a simpler variant is the -unixpw_nis option that
       works in environments where the encrypted passwords are readable,
       e.g. NIS.) The -ssl or -localhost + -stunnel options are enforced
       in this mode to prevent password sniffing. As a convenience, these
       requirements are lifted if a SSH tunnel can be deduced (but
       -localhost still applies.)
     * Coupling -unixpw with "-display WAIT:cmd=FINDDISPLAY" or "-display
       WAIT:cmd=FINDCREATEDISPLAY" provides a way to allow a user to
       login with their UNIX password and have their display connected to
       automatically. See the -svc and the -xdmsvc aliases.
     * Hooks are provided in the -unixpw_cmd and "-passwdfile
       cmd:,custom:..." options to allow you to supply your own
       authentication and password lookup programs.
     * x11vnc can be configured and built to not depend on X11 libraries
       "./configure --without-x" for -rawfb only operation (e.g. embedded
       linux console devices.)
     * The -rotate option enables you to rotate or reflect the screen
       before exporting via VNC. This is intended for use on handhelds
       and other devices where the rotation orientation is not "natural".
     * The "-ultrafilexfer" alias is provided and improved UltraVNC
       filetransfer rates have been achieved.
     * Under the "-connect_or_exit host" option x11vnc will exit
       immediately unless the reverse connection to host succeeds. The
       "-rfbport 0" option disables TCP listening for connections (useful
       for this mode.)
     * The "-rawfb rand" and "-rawfb none" options are useful for testing
       automation scripts, etc., without requiring a full desktop.
     * Reduced spewing of information at startup, use "-verbose" (also
       "-v") to turn it back on for debugging or if you are going to send
       me a problem report.

   Here are some Previous Release Notes
     _________________________________________________________________

    Some Notes:

   Both a client and a server:   It is sometimes confusing to people that
   x11vnc is both a client and a server at the same time. It is an X
   client because it connects to the running X server to do the screen
   polls. Think of it as a rather efficient "screenshot" program running
   continuously. It is a server in the sense that it is a VNC server that
   VNC viewers on the network can connect to and view the screen
   framebuffer it manages.

   When trying to debug problems, remember to think of both roles. E.g.
   "how is x11vnc connecting to the X server?", "how is the vncviewer
   connecting to x11vnc?", "what permits/restricts the connection?". Both
   links may have reachability, permission, and other issues.

   Network performance:   Whether you are using Xvnc or x11vnc it is
   always a good idea to have a solid background color instead of a
   pretty background image. Each and every re-exposure of the background
   must be resent over the network: better to have that background be a
   solid color that compresses very well compared to a photo image. (This
   is one place where the X protocol has an advantage over the VNC
   protocol.) I suggest using xsetroot, dtstyle or similar utility to set
   a solid background while using x11vnc. You can turn the pretty
   background image back on when you are using the display directly.
   Update: As of Feb/2005 x11vnc has the -solid [color] option that works
   on recent GNOME, KDE, and CDE and also on classic X (background image
   is on the root window.) Update: As of Oct/2007 x11vnc has the -ncache
   option that does a reasonable job caching the background (and other)
   pixmap data on the viewer side.

   I also find the TightVNC encoding gives the best response for my usage
   (Unix <-> Unix over cable modem.) One needs a tightvnc-aware vncviewer
   to take advantage of this encoding.

   TCP port issues:   Notice the lines
  18/07/2003 14:36:31 Autoprobing selected port 5900
  PORT=5900

   in the output. 5900 is the default VNC listening port (just like 6000
   is X11's default listening port.) Had port 5900 been taken by some
   other application, x11vnc would have next tried 5901. That would mean
   the viewer command above should be changed to vncviewer
   far-away.east:1. You can force the port with the "-rfbport NNNN"
   option where NNNN is the desired port number. If that port is already
   taken, x11vnc will exit immediately. The "-N" option will try to match
   the VNC display number to the X display.   (also see the "SunRay
   Gotcha" note below)

   Options:   x11vnc has (far too) many features that may be activated
   via its command line options. Useful options are, e.g., -scale to do
   server-side scaling, and -rfbauth passwd-file to use VNC password
   protection (the vncpasswd or storepasswd programs, or the x11vnc
   -storepasswd option can be used to create the password file.)

   Algorithm:   How does x11vnc do it? Rather brute-forcedly: it
   continuously polls the X11 framebuffer for changes using
   XShmGetImage(). When changes are discovered, it instructs libvncserver
   which rectangular regions of the framebuffer have changed, and
   libvncserver compresses the changes and sends them off to any
   connected VNC viewers. A number of applications do similar things,
   such as x0rfbserver, krfb, x0vncserver, vino. x11vnc uses a 32 x 32
   pixel tile model (the desktop is decomposed into roughly 1000 such
   tiles), where changed tiles are found by pseudo-randomly polling 1
   pixel tall horizontal scanlines separated vertically by 32 pixels.
   This is a surprisingly effective algorithm for finding changed
   regions. For keyboard and mouse user input the XTEST extension is used
   to pass the input events to the X server. To detect XBell "beeps" the
   XKEYBOARD extension is used. If available, the XFIXES extension is
   used to retrieve the current mouse cursor shape. Also, if available
   the X DAMAGE extension is used to receive hints from the X server
   where modified regions on the screen are. This greatly reduces the
   system load when not much is changing on the screen and also improves
   how quickly the screen is updated.

   Barbershop mirrors effect:   What if x11vnc is started up, and
   vncviewer is then started up on the same machine and displayed on the
   same display x11vnc is polling? One might "accidentally" do this when
   first testing out the programs. You get an interesting
   recursive/feedback effect where vncviewer images keep popping up each
   one contained in the previous one and slightly shifted a bit by the
   window manager decorations. There will be an even more interesting
   effect if -scale is used. Also, if the XKEYBOARD is supported and the
   XBell "beeps" once, you get an infinite loop of beeps going off.
   Although all of this is mildly exciting it is not much use: you will
   normally run and display the viewer on a different machine!
     _________________________________________________________________

    Sun Ray Notes:

   You can run x11vnc on your (connected or disconnected) SunRay session.
   Here are some notes on SunRay usage with x11vnc.

     _________________________________________________________________

    Limitations:

     * Due to the polling nature, some activities (opaque window moves,
       scrolling), can be pretty choppy/ragged and others (exposures of
       large areas) slow. Experiment with interacting a bit differently
       than you normally do to minimize the effects (e.g. do fullpage
       paging rather than line-by-line scrolling, and move windows in a
       single, quick motion.) Recent work has provided the
       -scrollcopyrect and -wireframe speedups using the CopyRect VNC
       encoding and other things, but they only speed up some activities,
       not all.
     * A rate limiting factor for x11vnc performance is that graphics
       hardware is optimized for writing, not reading (x11vnc reads the
       video framebuffer for the screen image data.) The difference can
       be a factor of 10 to 1000, and so it usually takes about 0.5-1 sec
       to read in the whole video hardware framebuffer (e.g. 5MB for
       1280x1024 at depth 24 with a read rate of 5-10MB/sec.) So whenever
       activity changes most of the screen (e.g. moving or iconifying a
       large window) there is a delay of 0.5-1 sec while x11vnc reads the
       changed regions in.
       A slow framebuffer read rate will often be the performance
       bottleneck on a fast LAN (whereas on slower links the reduced
       network bandwidth becomes the bottleneck.)
       Note: A quick way to get a 2X speedup of this for x11vnc is to
       switch your X server from depth 24 (32bpp) to depth 16 (16bpp.)
       You get a 4X speedup going to 8bpp, but the lack of color cells is
       usually unacceptable.
       To get a sense of the read and write speeds of your video card,
       you can run benchmarks like: "x11perf -getimage500",  "x11perf
       -putimage500",  "x11perf -shmput500" and for XFree86 displays with
       direct graphics access the "dga" command (press "b" to run the
       benchmark and then after a few seconds press "q" to quit.) Even
       this "dd if=/dev/fb0 of=/dev/null" often gives a good estimate.
       x11vnc also prints out its estimate:
  28/02/2009 11:11:07 Autoprobing TCP port
  28/02/2009 11:11:07 Autoprobing selected port 5900
  28/02/2009 11:11:08 fb read rate: 10 MB/sec
  28/02/2009 11:11:08 screen setup finished.
       We have seen a few cases where the hardware fb read speed is
       greater than 65 MB/sec: on high end graphics workstations from SGI
       and Sun, and also from a Linux user using nvidia proprietary
       drivers for his nvidia video card. Update 2008: thankfully, these
       sped up drivers are becoming more common on Linux and *BSD systems
       and that makes x11vnc run somewhat more quickly. Sometimes they
       have a read rate of over 400 MB/sec.
       On XFree86/Xorg it is actually possible to increase the
       framebuffer read speed considerably (10-100 times) by using the
       Shadow Framebuffer (a copy of the framebuffer is kept in main
       memory and this can be read much more quickly.) To do this one
       puts the line Option "ShadowFB" "true" in the Device section of
       the /etc/X11/XF86Config or /etc/X11/xorg.conf file. Note that this
       disables 2D acceleration at the physical display and so that might
       be unacceptable if one plays games, etc. on the machine's local
       display. Nevertheless this could be handy in some circumstances,
       e.g. if the slower speed while sitting at the physical display was
       acceptable (this seems to be true for most video cards these
       days.) Unfortunately it does not seem shadowfb can be turned on
       and off dynamically...
       Another amusing thing one can do is use Xvfb as the X server, e.g.
       "xinit $HOME/.xinitrc -- /usr/X11R6/bin/Xvfb :1 -screen 0
       1024x768x16" x11vnc can poll Xvfb efficiently via main memory.
       It's not exactly clear why one would want to do this instead of
       using vncserver/Xvnc, (perhaps to take advantage of an x11vnc
       feature, such as framebuffer scaling or built-in SSL encryption),
       but we mention it because it may be of use for special purpose
       applications. You may need to use the "-cc 4" option to force Xvfb
       to use a TrueColor visual instead of DirectColor. See also the
       description of the -create option that does all of this
       automatically for you (be sure to install the Xvfb package, e.g.
       apt-get install xvfb.)
       Also, a faster and more accurate way is to use the "dummy"
       Xorg/XFree86 device driver (or our Xdummy wrapper script.) See
       this FAQ for details.
     * Somewhat surprisingly, the X11 mouse (cursor) shape is write-only
       and cannot be queried from the X server. So traditionally in
       x11vnc the cursor shape stays fixed at an arrow. (see the "-cursor
       X" and "-cursor some" options, however, for a partial hack for the
       root window, etc.) However, on Solaris using the SUN_OVL overlay
       extension, x11vnc can show the correct mouse cursor when the
       -overlay option is also supplied. A similar thing is done on IRIX
       as well when -overlay is supplied.
       More generally, as of Dec/2004 x11vnc supports the new XFIXES
       extension (in Xorg and Solaris 10) to query the X server for the
       exact cursor shape, this works pretty well except that cursors
       with transparency (alpha channel) need to approximated to solid
       RGB values (some cursors look worse than others.)
     * Audio from applications is of course not redirected (separate
       redirectors do exist, e.g. esd, see the FAQ on this below.) The
       XBell() "beeps" will work if the X server supports the XKEYBOARD
       extension. (Note that on Solaris XKEYBOARD is disabled by default.
       Passing +kb to Xsun enables it.)
     * The scroll detection algorithm for the -scrollcopyrect option can
       give choppy or bunched up transient output and occasionally
       painting errors.
     * Using -threads can expose some bugs/crashes in libvncserver.

   Please feel free to contact me if you have any questions, problems, or
   comments about x11vnc, etc. Please be polite, thorough, and not
   demanding (sadly, the number of people contacting me that are rude and
   demanding is increasing dramatically.)