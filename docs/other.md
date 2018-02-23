





    Contributions:

   Q-132: Thanks for your program or for your help! Can I make a
   donation?

   Please do (any amount is appreciated; very few have donated) and thank
   you for your support! Click on the PayPal button below for more info.

   [x-click-but04.gif]-Submit
	
=======================================================================
http://www.karlrunge.com/x11vnc/chainingssh.html:


     _________________________________________________________________

   Chaining ssh's: Note that for use of a ssh gateway and -L redirection
   to an internal host (e.g. "-L 5900:otherhost:5900") the VNC traffic
   inside the firewall is not encrypted and you have to manually log into
   otherhost to start x11vnc. Kyle Amon shows a method where you chain
   two ssh's together that encrypts all network traffic and also
   automatically starts up x11vnc on the internal workstation:
#!/bin/sh
#
gateway="example.com"   # or "user@example.com"
host="labyrinth"        # or "user@hostname"
user="kyle"

# Need to sleep long enough for all of the passwords and x11vnc to start up.
# The </dev/null below makes the vncviewer prompt for passwd via popup window.
#
(sleep 10; vncviewer -encodings "copyrect tight zrle zlib hextile" \
    localhost:0 </dev/null >/dev/null) &

# Chain the vnc connection thru 2 ssh's, and connect x11vnc to user's display:
#
exec /usr/bin/ssh -t -L 5900:localhost:5900 $gateway \
     /usr/bin/ssh -t -L 5900:localhost:5900 $host \
     sudo /usr/bin/x11vnc -localhost -auth /home/$user/.Xauthority \
         -rfbauth .vnc/passwd -display :0

   Also note the use of sudo(1) to switch to root so that the different
   user's .Xauthority file can be accessed. See the visudo(8) manpage for
   details on how to set this up (remove the sudo if you do not want to
   do this). One can also chain together ssh's for reverse connections
   with vncviewers using the -listen option. For this case -R would
   replace the -L (and 5500 the 5900, see the #2 example script above).
   If the gateway machine's sshd is configured with GatewayPorts=no (the
   default) then the double chaining of "ssh -R ..." will be required for
   reverse connections to work.
	
=======================================================================
http://www.karlrunge.com/x11vnc/miscbuild.html:


     _________________________________________________________________

   Misc. Build problems:   We collect here rare build problems some users
   have reported and the corresponding workarounds. See also the FAQ's on
   building.
     _________________________________________________________________

   ENV parameter: One user had a problem where the build script below was
   failing because his work environment had the ENV variable set to a
   script that was resetting his PATH so that gcc could no longer be
   found. Make sure you do not have any ENV or BASH_ENV in your
   environment doing things like that. Typing "unset ENV", etc. before
   configuring and building should clear it.
     _________________________________________________________________

   Bash xpg: One user had his bash shell compiled with
   --enable-xpg-echo-default that causes some strange behavior with
   things like echo "\\1 ..." the configure script executes. In
   particular instead of getting "\1" the non-printable character "^A" is
   produced, and causes failures at compile time like:
  ../rfb/rfbconfig.h:9:22: warning: extra tokens at end of #ifndef directive

   The workaround is to configure like this:
  env CONFIG_SHELL=/bin/sh /bin/sh ./configure

   i.e. avoid using the bash with the misbehavior. A bug has been filed
   against autoconf to guard against this.
     _________________________________________________________________

   AIX: one user had to add the "X11.adt" package to AIX to get build
   header files like XShm.h, etc.
     _________________________________________________________________

   Ubuntu Feisty Fawn 7.04: In May/2007 one user said he needed to add
   these packages to compile x11vnc on that Linux distro and version:
  apt-get install build-essential make bin86 libjpeg62-dev libssl-dev libxtst-d
ev

   Note that Ubuntu is based on Debian, so perhaps this is the list
   needed on Debian (testing?) as well. To build in Avahi (mDNS service
   advertising) support it would appear that libavahi-client-dev is
   needed as well.
     _________________________________________________________________

   Exceedingly slow compilation: x11vnc has a couple of files which
   contain very large "case statements" (over 100 cases) that on some
   platforms can take a very long time to compile (in extreme cases over
   an hour). However on 32bit Linux with intel/amd processor and gcc
   these files usually take less than 10 seconds to compile. For 64bit
   systems using gcc the problem appears to be much worse.

   The two files with the large number of cases, remote.c and x11vnc.c,
   have no real need to be optimized (the code is used only very
   infrequently). So it is fine to supply "-O0" (disables optimization)
   to CFLAGS when compiling them. However, it is tricky with
   autoconf/automake to do this (especially since both the compiler and
   make versions have a big effect).

   So if the compile times are getting too long for you for these two
   files you will need to manually change some things. First, run
   configure and when it has finished, edit the generated file
   x11vnc/Makefile and put these lines at the very top:
x11vnc-x11vnc.o :  CFLAGS += -O0
x11vnc-remote.o :  CFLAGS += -O0

   Those lines assume gnu make (gmake) is being used. If you are using
   another make, say Solaris make, insert these instead:
x11vnc-x11vnc.o := CFLAGS += -O0
x11vnc-remote.o := CFLAGS += -O0

   You could write a build shell script that modified the Makefile this
   way before running make.

   The "-O0" (note it is "capital Oh" followed by "zero") assumes the gcc
   compiler. If you are using a different compiler you will need to find
   the command line option to disable optimization, or otherwise have the
   lines set CFLAGS to the empty string.
     _________________________________________________________________

   Broken Thread Local Storage on SuSE 9.2: Starting with x11vnc 0.9.8
   the bundled libvncserver uses the __thread keyword to make some of the
   encodings (i.e. tight) thread safe (multiple VNC clients can be using
   tight at the same time in x11vnc -threads mode.) Evidently on the old
   SuSE 9.2 system the compiler does not support the thread local storage
   properly. Here is an example build failure:
tight.c:1126: error: unrecognizable insn:
(insn:HI 11 10 13 0 (nil) (set (reg/f:SI 59)
        (const:SI (plus:SI (symbol_ref:SI ("%lpalette"))
                (const_int 2048 [0x800])))) -1 (nil)
    (expr_list:REG_EQUAL (const:SI (plus:SI (symbol_ref:SI ("%lpalette"))
                (const_int 2048 [0x800])))
        (nil)))
tight.c:1126: internal compiler error: in extract_insn, at recog.c:2175
Please submit a full bug report,
with preprocessed source if appropriate.
See URL:http://www.suse.de/feedback for instructions.

   The workaround is to disable thread local storage at configure time
   like this:
env CPPFLAGS="-DTLS=''" ./configure

   and then build it.
     _________________________________________________________________
	
=======================================================================
http://www.karlrunge.com/x11vnc/sunray.html:


    Sun Ray Notes:

   You can run x11vnc on your (connected or disconnected) SunRay session
   (Please remember to use settings like -wait 200, -sb 15, and not
   running a screensaver animation (blank instead) to avoid being a
   resource hog! x11vnc does induce a lot of memory I/O from polling the
   X server. It also helps to have a solid background color, e.g.
   -solid).

   News: Sun Ray Remote Control Toolkit: See the nice set of tools in the
   Sun Ray Remote Control Toolkit that launch x11vnc automatically for
   you for certain usage modes.

   You have to know the name of the machine your SunRay session X server
   is running on (so you can ssh into it and start x11vnc). You also need
   to know the X11 DISPLAY number for the session: on a SunRay it could
   be a large number, e.g. :137, since there are many people with X
   sessions (Xsun processes) on the same machine. If you don't know it,
   you can get it by running who(1) in a shell on the SunRay server and
   looking for the dtlocal entry with your username (and if you don't
   even know which server machine has your session, you could login to
   all possible ones looking at the who output for your username...).

   I put some code in my ~/.dtprofile script that stores $DISPLAY
   (including the hostname) in a ~/.sunray_current file at session
   startup (and deletes it when the X session ends) to make it easy to
   get at the hostname and X11 display number info for my current X
   sessions when I ssh in and am about to start x11vnc.

   SunRay Gotcha #1:   Note that even though your SunRay X11 DISPLAY is
   something like :137, x11vnc still tries for port 5900 as its listening
   port if it can get it, in which case the VNC display (i.e. the
   information you supply to the VNC viewer) is something like
   sunray-server:0   (note the :0 corresponding to port 5900, it is not
   :137). If it cannot get 5900, it tries for 5901, and so on. You can
   also try to force the port (and thereby the VNC display) using the
   -rfbport NNNN option.

   Especially on a busy Sun Ray server it is often difficult to find free
   ports for both VNC and the HTTP Java applet server to listen on. This
   script, vnc_findports may be of use for doing this automatically. It
   suggests x11vnc command line options based on netstat output that
   lists the occupied ports. It is even more difficult to start
   vncserver/Xvnc on a busy Sun Ray because then 3 ports (HTTP, VNC, and
   X11), all separated by 100 are needed! This script, findvncports may
   be helpful as well. Both scripts start at VNC display :10 and work
   their way up.

   SunRay Gotcha #2:   If you get an error like:
        shmget(tile) failed.
        shmget: No space left on device

   when starting up x11vnc that most likely means all the shared memory
   (shm) slots are filled up on your machine. The Solaris default is only
   100, and that can get filled up in a week or so on a SunRay server
   with lots of users. If the shm slot is orphaned (e.g. creator process
   dies) the slot is not reclaimed. You can view the shm slots with the
   "ipcs -mA" command. If there are about 100 then you've probably hit
   this problem. They can be cleaned out (by the owner or by root) using
   the ipcrm command. I wrote a script shm_clear that finds the orphans
   and lists or removes them. Longer term, have your SunRay sysadmin add
   something like this to /etc/system:
        set shmsys:shminfo_shmmax = 0x2000000
        set shmsys:shminfo_shmmni = 0x1000

   SunRay Gotcha #3:   Some SunRay installations have implemented
   suspending certain applications when a SunRay session is in a
   disconnected state (e.g. Java Badge pulled out, utdetach, etc). This
   is a good thing because it limits hoggy or runaway apps from wasting
   the shared CPU resource. Think how much CPU and memory I/O is wasted
   by a bunch of Firefox windows running worthless Flash animations while
   your session is disconnected!

   So some sites have implemented scripts to suspend (e.g. kill -STOP)
   certain apps when your badge is removed from the SunRay terminal. When
   you reattach, it kill -CONT them. This causes problems for viewing the
   detached SunRay session via x11vnc: those suspended apps will not
   respond (their windows will be blank or otherwise inactive).

   What to do? Well, since you are going to be using the application you
   might as well unfreeze it rather than starting up a 2nd instance. Here
   is one way to do it using the kill -CONT mechanism:
   kill -CONT `ps -ealf | grep ' T ' | grep $LOGNAME | awk '{print $4}'`

   If you want to be a good citizen and re-freeze them before you exit
   x11vnc this script could be of use:
#!/bin/sh
#
# kill -STOP/-CONT script for x11vnc (or other) SunRay usage ("freezes"
# certain apps from hogging resources when disconnected).
#
# Put here a pattern that matches the apps that are frozen:
#
appmatch="java_vm|jre|netscape-bin|firefox-bin|realplay|acroread|mozilla-bin"

if [ "X$1" = "Xfreeze" ]; then
        pkill -STOP -U $LOGNAME "$appmatch"
elif [ "X$1" = "Xthaw" ]; then
        pkill -CONT -U $LOGNAME "$appmatch"

elif [ "$RFB_MODE" = "afteraccept" -a "$RFB_STATE" = "NORMAL" ]; then
        # a valid x11vnc login.
        if [ "$RFB_CLIENT_COUNT" = "1" ]; then
                # only one client present.
                pkill -CONT -U $LOGNAME "$appmatch"
        fi
elif [ "$RFB_MODE" = "gone" -a "$RFB_STATE" = "NORMAL" ]; then
        # a valid x11vnc login.
        if [ "$RFB_CLIENT_COUNT" = "0" ]; then
                # last client present has just left.
                pkill -STOP -U $LOGNAME "$appmatch"
        fi
fi
exit 0

   If you called the script "goodcitizen" you could type "goodcitizen
   thaw" to unfreeze them, and then "goodcitizen freeze" to refreeze
   them. One could also use these x11vnc options "-afteraccept
   goodcitizen -gone goodcitizen" to do it automatically.

   SunRay Gotcha #4:   Recent versions of the Sun Ray Server Software
   SRSS (seems to be version 3.0 or 3.1) have a "misfeature" that when
   the session is disconnected (i.e. badge/smartcard out) the screen
   locker (xscreensaver) will freeze the X server just when the "Enter
   Password" dialog box appears. So you cannot unlock the screen remotely
   via x11vnc!

   Update: please see Bob Doolittle's detailed description of the this
   issue at the bottom of this section.

   Here "freeze" means "stop other X clients from inserting keyboard and
   mouse input and from viewing the current contents of the screen". Or
   something like that; the upshot is x11vnc can't do its normal thing.

   There are several workarounds for this.

   1) The easiest one by far is to put these lines in your
   $HOME/.dtprofile file:
SUN_SUNRAY_UTXLOCK_PREF="/usr/openwin/bin/xlock -mode blank"
export SUN_SUNRAY_UTXLOCK_PREF

   One might argue that xlock isn't particularly "pretty". (Just IMHO,
   but if something like this not being pretty actually gets in the way
   of your work I think some introspection may be in order. :-)

   2) The problem has been traced to the pam_sunray.so PAM module.
   Evidently xscreensaver invokes this pam module and it communicates
   with utsessiond who in turn instructs the Xsun server to not process
   any synthetic mouse/keyboard input or to update the screen
   framebuffer. It is not clear if this is by design (security?) or
   something else.

   In any event, the problem can be avoided, somewhat drastically, by
   commenting out the corresponding line in /etc/pam.conf:
#xscreensaver auth sufficient /opt/SUNWut/lib/pam_sunray.so syncondisplay

   Leave the other xscreensaver pam authentication lines unchanged. The
   dtsession-SunRay line may also need to be commented out to avoid the
   problem for CDE sessions. N.B. it is possible the application of a
   SSRS patch, etc, may re-enable that /etc/pam.conf line. It may be
   difficult to convince a sysadmin to make this change.

   3) A more forceful way is to kill the xscreensaver process from a
   shell prompt whenever you connect via x11vnc and the screen is in a
   locked state:
pkill -U $LOGNAME '^xscreensaver$'

   And then after you are in be sure to restart it by typing something
   like:
xscreensaver &

   You may want to avoid restarting it until you are about to disconnect
   your VNC viewer (since if it locks the screen while you are working
   you'll be stuck again).

   3') The above idea can be done a bit more cleanly by having x11vnc do
   it. Suppose we called the following script xss_killer:
#!/bin/sh
#
# xss_killer: kill xscreensaver after a valid x11vnc client logs in.
#             Restart xscreensaver and lock it when the last client
#             disconnects.

PATH=/usr/openwin/bin:/usr/bin:$PATH
export PATH

if [ "$RFB_MODE" = "afteraccept" -a "$RFB_STATE" = "NORMAL" ]; then
        # a valid x11vnc login.
        if [ "$RFB_CLIENT_COUNT" = "1" ]; then
                # only one client present.
                pkill -U $LOGNAME '^xscreensaver$'
                pkill -KILL -U $LOGNAME -f xscreensaver/hacks
        fi
elif [ "$RFB_MODE" = "gone" -a "$RFB_STATE" = "NORMAL" ]; then
        # a valid x11vnc login.
        if [ "$RFB_CLIENT_COUNT" = "0" ]; then
                # last client present has just left.
                xscreensaver -nosplash &
                sleep 1
                xscreensaver-command -lock &
        fi
fi

   Then we would run x11vnc with these options: "-afteraccept xss_killer
   -gone xss_killer". The -afteraccept option (introduced in version 0.8)
   is used to run a command after a vncviewer has successfully logged in
   (note that this is a VNC login, not a Unix login, so you may not want
   to do this if you are really paranoid...)

   Note if you use the above script and also plan to Ctrl-C (SIGINT)
   x11vnc you have to run the xscreensaver in a new process group to
   avoid killing it as well. One way to do this is via this kludge:
perl -e 'setpgrp(0,0); exec "xscreensaver -nosplash &"'

   in the above script.

   4) There appears to be a bug in pam_sunray.so in that it doesn't seem
   to honor the convention that, say, DISPLAY=unix:3 means to use Unix
   sockets to connect to display 3 on the local machine (this is a bit
   faster than TCP sockets). Rather, it thinks the display is a non-local
   one to a machine named "unix" (that usually does not resolve to an IP
   address).

   Amusingly, this can be used to bypass the pam_sunray.so blocking of
   Xsun that prevents one from unlocking the screen remotely via x11vnc.
   One could put something like this in $HOME/.dtprofile to kill any
   existing xscreensavers and then start up a fresh xscreensaver using
   DISPLAY=unix:N
# stop/kill any running xscreensavers (probably not running yet, but to be sure
)
xscreensaver-command -exit
pkill -U $LOGNAME '^xscreensaver$'
env DISPLAY=`echo $DISPLAY | sed -e 's/^.*:/unix:/'` xscreensaver &


   Important: Note that all of the above workarounds side-step the
   pam_sunray.so PAM module in one way or another. You'll need to see if
   that is appropriate for your site's SunRay / smartcard usage. Also,
   these hacks may break other things and so you may want to test various
   scenarios carefully. E.g. check corner cases like XDMCP/dtremote,
   NSCM, etc.


   Update May 2008: Here is a useful description of this issue from Bob
   Doolittle who is a developer for Sun Ray at Sun. I don't have the time
   to digest and distill it and then adjust the above methods to provide
   a clearer description, so I just include below the description he sent
   me with the hope that it will help some users:

     In SRSS 4.0 and earlier, the purpose of pam_sunray.so in the "auth"
     PAM stack of screensavers is to enable NSCM (and, although this is
     much less commonly used, "SC", which is configured when 3rd-party
     software is installed to allow smartcards to be used as part of the
     authentication process) to work. It should have no effect with
     smartcards. Currently, however, it does block the PAM stack for all
     sessions, which causes xscreensaver, when it locks a disconnected
     session, to not process any mouse or keyboard events as you
     describe (unless xscreensaver does an X server grab, however, other
     applications should still be able to draw in the session although
     xscreensaver may be playing tricks like putting a black window on
     top of everything). In both of the NSCM and SC models,
     authentication occurs in a separate session before SRSS will
     reconnect to the user session, in which case pam_sunray.so causes
     xscreensaver to just unlock the screen without prompting the user
     to enter their password again. To do this, pam_sunray.so has to
     block until the session becomes reconnected, so it can query SRSS
     at that time to determine whether the user has already
     authenticated or not. In SRSS 4.0 and earlier releases,
     pam_sunray.so could have been optimized to not block smartcard
     sessions, although since the session is disconnected this typically
     isn't important (except in the x11vnc case, as you've observed).

     In SRSS 4.1, however, for increased security the out-of-session
     authentication model has been extended to *all* session types, so
     pam_sunray.so will be required in all cases unless users are
     willing to authenticate twice upon hotdesking (e.g. when their card
     is inserted). In future, we may do away with pam_sunray.so, and in
     fact with any traditional screen locker in the user session, since
     SRSS itself will be providing better security than a screen locker
     running entirely within the user's X session is capable of
     providing.

     Your trick of setting DISPLAY to unix:DPY will effectively disable
     pam_sunray.so (I'm not sure I'd call that a bug - you're going out
     of your way to do something that wouldn't occur in the normal
     course of events, and really provides no useful value other than to
     tickle this behavior in pam_sunray.so). This will mean that, in
     SRSS 4.0 and earlier releases, users will be prompted for their
     passwords twice when reconnecting to their sessions for NSCM and SC
     session types. In 4.1, disabling pam_sunray.so in this way will
     cause this double-authentication to occur for *all* sessions,
     including simple smartcard sessions. Users may be willing to pay
     that price in order to be able to use x11vnc in disconnected
     sessions. I like this hack, personally. It's a little less
     convenient than some of the other approaches you describe, but it's
     lighter-weight and more secure than most of the other approaches,
     and provides the value of being able to use x11vnc in locked
     sessions.

     Here are some other minor notes: - I wouldn't recommend storing
     your display in your .dtprofile, unless you're willing to live with
     a single session at a time. Personally, I often find myself using
     several sessions, in several FoGs, for short periods of time so
     this would certainly break. IMO it's pretty easy to use $DISPLAY to
     do what you want on the fly, as needed, so I don't think the price
     of breaking multiple-session functionality would be worth the
     convenience, to me at least. Here's some ksh/bash syntax to extract
     the hostname and display number on the fly which you may find
     useful:
HOSTNAME=${DISPLAY%:*}
FULLDPY=${DISPLAY#*:}
DPYNUM=${FULLDPY%.*}

     A final note may give you some insight into other clever hacks in
     this area: - Check out utaction. It's a very handy little utility
     that can be run as a daemon in the user session which will invoke a
     specified command upon session connects and/or disconnects.
     Personally, I start one up in my .dtprofile as follows:
utaction -c $HOME/.srconnectrc -d $HOME/.srdisconnectrc &

     This then allows me to construct a .srconnectrc script containing
     useful commands I'd like to have run every time I insert my
     smartcard, and a .srdisconnectrc script of commands to be run every
     time I remove my smartcard (or, connect/disconnect to my session
     via NSCM or SC). This can be used for things like notifying a chat
     client of away status, as well as some of the hacks you've
     described on your page such as freeze/unfreeze, or perhaps to
     terminate an xscreensaver and start up a new one with the unix:DPY
     $DISPLAY specification as you describe (although it probably makes
     most sense to do this at login time, as opposed to every connect or
     disconnect event).
	
=======================================================================
http://www.karlrunge.com/x11vnc/ssl.html:


     _________________________________________________________________

   Notes on x11vnc SSL Certificates and Key Management:

   The simplest scheme ("x11vnc -ssl TMP") is where x11vnc generates a
   temporary, self-signed certificate each time (automatically using
   openssl(1)) and the VNC viewer client accepts the certificate without
   question (e.g. user clicks "Yes" in a dialog box. Perhaps the dialog
   allows them to view the certificate too). Also note stunnel's default
   is to quietly accept all certificates.

   The encryption this provides protects against all passive sniffing of
   the VNC traffic and passwords on the network and so it is quite good,
   but it does not prevent a Man-In-The-Middle active attack: e.g. an
   attacker intercepts the VNC client stream and sends it his own Public
   key for SSL negotiation (pretending to be the server). Then it makes a
   connection to SSL x11vnc itself and forwards the data back and forth.
   He can see all the traffic and modify it as well.

   Most people don't seem to worry about Man-In-The-Middle attacks these
   days; they are more concerned about passive sniffing of passwords,
   etc. Perhaps someday that will change if attack tools are used more
   widely to perform the attack. NOTE: There are hacker tools like
   dsniff/webmitm and cain that implement SSL Man-In-The-Middle attacks.
   They all rely on the client not bothering to check that the cert is
   valid.

   If you are not worried about Man-In-The-Middle attacks you do not have
   to read the techniques described in the rest of this document.

   To prevent Man-In-The-Middle attacks, certificates must somehow be
   verified. This requires the VNC client side have some piece of
   information that can be used to verify the SSL x11vnc server.
   Alternatively, although rarely done, x11vnc can verify VNC Clients'
   certificates, see the -sslverify option that is discussed below.

   There are a number of ways to have the client authenticate the SSL
   x11vnc server. The quickest way perhaps would be to copy (safely) the
   certificate x11vnc prints out:
26/03/2006 21:12:00 Creating a temporary, self-signed PEM certificate...
...
-----BEGIN CERTIFICATE-----
MIIC4TCCAkqgAwIBAgIJAMnwCaOjvEKaMA0GCSqGSIb3DQEBBAUAMIGmMQswCQYD
VQQGEwJBVTEOMAwGA1UEBxMFTGludXgxITAfBgNVBAsTGGFuZ2VsYS0xMTQzNDI1
NTIwLjQxMTE2OTEPMA0GA1UEChMGeDExdm5jMS4wLAYDVQQDEyV4MTF2bmMtU0VM
(more lines) ...
-----END CERTIFICATE-----

   to the client machine(s) and have the client's SSL machinery (e.g.
   stunnel, Web Browser, or Java plugin) import the certificate. That way
   when the connection to x11vnc is made the client can verify that is it
   the desired server on the other side of the SSL connection.

   So, for example suppose the user is using the SSL enabled Java VNC
   Viewer and has incorporated the x11vnc certificate into his Web
   browser on the viewing side. If he gets a dialog that the certificate
   is not verified he knows something is wrong. It may be a
   Man-In-The-Middle attack, but more likely x11vnc certificate has
   changed or expired or his browser was reinstalled and/or lost the
   certificate, etc, etc.

   As another example, if the user was using stunnel with his VNC viewer
   (this is mentioned in this FAQ), e.g. STUNNEL.EXE on Windows, then he
   would have to set the "CAfile = path-to-the-cert" and "verify = 2"
   options in the stunnel.conf file before starting up the tunnel. If a
   x11vnc certificate cannot be verified, stunnel will drop the
   connection (and print a failure message in its log file).

   A third example, using the VNC viewer on Unix with stunnel the wrapper
   script can be used this way: "ss_vncviewer -verify ./x11vnc.crt
   far-away.east:0" where ./x11vnc.crt is the copied certificate x11vnc
   printed out.

   As fourth example, our SSVNC enhanced tightvnc viewer can also use
   these certificate files for server authentication. You can load them
   via the SSVNC 'Certs...' dialog and set 'ServerCert' to the
   certificate file you safely copied there.

   Note that in principle the copying of the certificate to the client
   machine(s) itself could be altered by a Man-In-The-Middle attack! You
   can't win; it is very difficult to be completely secure. It is
   unlikely the attacker could predict how you were going to send it
   unless you had, say, done it many times before the same way. SSH is a
   very good way to send it (but of course it too depends on public keys
   being sent unaltered between the two machines!).

   If you are really paranoid, I'm sure you'll figure out a really good
   way to transport the certificates. See the Certificate Authority
   scheme below for a way to make this easier (you just have to do it
   once).

     _________________________________________________________________

   Saving SSL certificates and keys:

   Now, it would be very inconvenient to copy the new temporary
   certificate every time x11vnc is run in SSL mode. So for convenience
   there is the "SAVE" keyword to instruct x11vnc to save the certificate
   it creates:
  x11vnc -ssl SAVE -display :0 ...

   This behavior is now the default, you must use "TMP" for a temporary
   one. It will save the certificate and private key in these files:
  ~/.vnc/certs/server.crt
  ~/.vnc/certs/server.pem

   The ".crt" file contains only the certificate and should be safely
   copied to the VNC Viewer machine(s) that will be authenticating the
   x11vnc server. The ".pem" file contains both the certificate and the
   private key and should be kept secret. (If you don't like the default
   location ~/.vnc/certs, e.g. it is on an NFS share and you are worried
   about local network sniffing, use the -ssldir dir option to point to a
   different directory.)

   So the next time you run "x11vnc -ssl SAVE ..." it will read the
   server.pem file directly instead of creating a new one.

   You can manage multiple SSL x11vnc server keys in this simple way by
   using:
  x11vnc -ssl SAVE-key2 -display :0 ...

   etc, where you put whatever name you choose for the key after "SAVE-".
   E.g. "-ssl SAVE-fred".

   Also, if you want to be prompted to possibly change the made up names,
   etc. that x11vnc creates (e.g. "x11vnc-SELF-SIGNED-CERT-7762" for the
   CommonName) for the certificates distinguished name (DN), then use
   "x11vnc -ssl SAVE_PROMPT ...", "x11vnc -ssl SAVE_PROMPT-fred ..." etc.
   when you create the key the first time.

   Tip: when prompting, if you choose the CommonName entry to be the full
   internet hostname of the machine the clients will be connecting to
   then that will avoid an annoying dialog box in their Web browsers that
   warn that the CommonName doesn't match the hostname.

     _________________________________________________________________

   Passphrases for server keys:

   Well, since now with the "SAVE" keyword the certificate and key will
   be longer lived, one can next worry about somebody stealing the
   private key and pretending to be the x11vnc server! How to guard
   against this?

   The first is that the file is created with perms 600 (i.e. -rw-------)
   to make it harder for an untrusted user to copy the file. A better way
   is to also encrypt the private key with a passphrase. You are prompted
   whether you want to do this or not when the key is first created under
   "-ssl SAVE" mode ("Protect key with a passphrase? y/n"). It is
   suggested that you use a passphrase. The inconvenience is every time
   you run "x11vnc -ssl SAVE ..." you will need to supply the passphrase
   to access the private key:
  06/04/2006 11:39:11 using PEM /home/runge/.vnc/certs/server.pem  0.000s

  A passphrase is needed to unlock an OpenSSL private key (PEM file).
  Enter passphrase>

   before x11vnc can continue.

     _________________________________________________________________

   Being your own Certificate Authority:

   A very sophisticated way that scales well if the number of users is
   large is to use a Certificate Authority (CA) whose public certificate
   is available to all of the VNC clients and whose private key has been
   used to digitally sign the x11vnc server certificate(s).

   The idea is as follows:
     * A special CA cert and key is generated.
     * Its private key is always protected by a good passphrase since it
       is only used for signing.
     * The CA cert is (safely) distributed to all machines where VNC
       clients will run.
     * One or more x11vnc server certs and keys are generated.
     * The x11vnc server cert is signed with the CA private key.
     * x11vnc is run using the server key. (e.g. "-ssl SAVE")
     * VNC clients (viewers) can now authenticate the x11vnc server
       because they have the CA certificate.

   The advantage is the CA cert only needs to be distributed once to the
   various machines, that can be done even before x11vnc server certs are
   generated.

   As above, it is important the CA private key and the x11vnc server key
   are kept secret, otherwise someone could steal them and pretend to be
   the CA or the x11vnc server if they copied the key. It is recommended
   that the x11vnc server keys are also protected via a passphrase (see
   the previous section).

   Optionally, VNC viewer certs and keys could also be generated to
   enable the x11vnc server to authenticate each client. This is not
   normally done (usually a simple viewer password scheme is used), but
   this can be useful in some situations. These optional steps go like
   this:
     * One or more VNC client certs and keys are generated.
     * These VNC client certs are signed with the CA private key.
     * The VNC client certs+keys are safely distributed to the
       corresponding client machines.
     * x11vnc is told to verify clients by using the CA cert. (e.g.
       "-sslverify CA")
     * When VNC clients (viewers) connect, they must authenticate
       themselves to x11vnc by using their client key.

   Again, it is a good idea if the client private keys are protected with
   a passphrase, otherwise if stolen they could be used to gain access to
   the x11vnc server. Once distributed to the client machines, there is
   no need to keep the client key on the CA machine that generated and
   signed it. You can keep the client certs if you like because they are
   public.

     _________________________________________________________________

   How to do the above CA steps with x11vnc:

   Some utility commands are provided to ease the cert+key creation,
   signing, and management: -sslGenCA, -sslGenCert, -sslDelCert,
   -sslEncKey, -sslCertInfo. They basically run the openssl(1) command
   for you to manage the certs/keys. It is required that openssl(1) is
   installed on the machine and available in PATH. All commands can be
   pointed to an alternate toplevel certificate directory via the -ssldir
   option if you don't want to use the default ~/.vnc/certs.

   1) To generate your Certificate Authority (CA) cert and key run this:
  x11vnc -sslGenCA

   Follow the prompts, you can modify any informational strings you care
   to. You will also be required to encrypt the CA private key with a
   passphrase. This generates these files:
  ~/.vnc/certs/CA/cacert.pem             (the CA public certificate)
  ~/.vnc/certs/CA/private/cakey.pem      (the encrypted CA private key)

   If you want to use a different directory use -ssldir It must supplied
   with all subsequent SSL utility options to point them to the correct
   directory.

   2) To generate a signed x11vnc server cert and key run this:
  x11vnc -sslGenCert server

   As with the CA generation, follow the prompts and you can modify any
   informational strings that you care to. This will create the files:
  ~/.vnc/certs/server.crt             (the server public certificate)
  ~/.vnc/certs/server.pem             (the server private key + public cert)

   It is recommended to protect the server private key with a passphrase
   (you will be prompted whether you want to). You will need to provide
   it whenever you start x11vnc using this key.

   3) Start up x11vnc using this server key:
  x11vnc -ssl SAVE -display :0 ...

   (SAVE corresponds to server.pem, see -sslGenCert server somename info
   on creating additional server keys, server-somename.crt ...)

   4) Next, safely copy the CA certificate to the VNC viewer (client)
   machine(s). Perhaps:
  scp ~/.vnc/CA/cacert.pem clientmachine:.

   5) Then the tricky part, make it so the SSL VNC Viewer uses this
   certificate! There are a number of ways this might be done, it depends
   on what your client and/or SSL tunnel is. Some examples:

   For the SSL Java VNC viewer supplied with x11vnc in
   classes/ssl/VncViewer.jar or classes/ssl/SignedVncViewer.jar:
     * Import the cacert.pem cert into your Web Browser (e.g. Edit ->
       Preferences -> Privacy & Security -> Manage Certificates ->
       WebSites -> Import)
     * Or Import the cacert.pem cert into your Java Plugin (e.g. run
       ControlPanel, then Security -> Certificates -> Secure Site ->
       Import)

   When importing, one would give the browser/java-plugin the path to the
   copied cacert.pem file in some dialog. Note that the Web browser or
   Java plugin is used for the server authentication. If the user gets a
   "Site not verified" message while connecting he should investigate
   further.

   For the use of stunnel (e.g. on Windows) one would add this to the
   stunnel.conf:
  # stunnel.conf:
  client = yes
  options = ALL
  CAfile = /path/to/cacert.pem          # or maybe C:\path\to\cacert.pem
  [myvncssl]
  accept = 5901
  connect = far-away.east:5900

   (then point the VNC viewer to localhost:1).

   Here is an example for the Unix stunnel wrapper script ss_vncviewer in
   our SSVNC package:
  ss_vncviewer -verify ./cacert.pem far-away.east:0

   Our SSVNC enhanced tightvnc viewer GUI can also use the certificate
   file for server authentication. You can load it via the SSVNC
   'Certs...' dialog and set 'ServerCert' to the cacert.pem file you
   safely copied there.

     _________________________________________________________________

   Tricks for server keys:

   To create additional x11vnc server keys do something like this:
  x11vnc -sslGenCert server myotherkey

   and use it this way:
  x11vnc -ssl SAVE-myotherkey ...

   The files will be ~/.vnc/certs/server-myotherkey.{crt,pem}

   You can also create a self-signed server key:
  x11vnc -sslGenCert server self:third_key

   and use it this way:
  x11vnc -ssl SAVE-self:third_key ...

   This key is not signed by your CA. This can be handy to have a key set
   separate from your CA when you do not want to create a 2nd CA
   cert+key.

     _________________________________________________________________

   Using external CA's:

   You don't have to use your own CA cert+key, you can use a third
   party's instead. Perhaps you have a company-wide CA or you can even
   have your x11vnc certificate signed by a professional CA (e.g.
   www.thawte.com or www.verisign.com or perhaps the free certificate
   service www.startcom.org or www.cacert.org).

   The advantage to doing this is that the VNC client machines will
   already have the CA certificates installed and you don't have to
   install it on each machine.

   To generate an x11vnc server cert+key this way you should generate a
   "request" for a certicate signing something like this (we use the name
   "external" in this example, it could be anything you want):
  x11vnc -sslGenCert server req:external

   This will create the request file:
  ~/.vnc/certs/server-req:external.req

   Which you should send to the external CA. When you get the signed
   certificate back from them, save it in the file:
  ~/.vnc/certs/server-req:external.crt

   and create the .pem this way:
  mv  ~/.vnc/certs/server-req:external.key    ~/.vnc/certs/server-req:external.
pem
  chmod 600 ~/.vnc/certs/server-req:external.pem
  cat ~/.vnc/certs/server-req:external.crt >> ~/.vnc/certs/server-req:external.
pem

   You also rename the two files (.crt and .pem) to have a shorter
   basename if you like. E.g.:
  mv  ~/.vnc/certs/server-req:external.pem  ~/.vnc/certs/server-ext.pem
  mv  ~/.vnc/certs/server-req:external.crt  ~/.vnc/certs/server-ext.crt

   and the use via "x11vnc -ssl SAVE-ext ...", etc.

   On the viewer side make sure the external CA's certificate is
   installed an available for the VNC viewer software you plan to use.

     _________________________________________________________________

   Using Client Keys for Authentication:

   You can optionally create certs+keys for your VNC client machines as
   well. After distributing them to the client machines you can have
   x11vnc verify the clients using SSL. Here is how to do this:

  x11vnc -sslGenCert client dilbert
  x11vnc -sslGenCert client wally
  x11vnc -sslGenCert client alice
  ...

   As usual, follow the prompts if you want to change any of the info
   field values. As always, it is a good idea (although inconvenient) to
   protect the private keys with a passphrase. These files are created:
  ~/.vnc/certs/clients/dilbert.crt
  ~/.vnc/certs/clients/dilbert.pem
  ...

   Note that these are kept in a clients subdirectory.

   Next, safely copy the .pem files to each corresponding client machine
   and incorporate them into the VNC viewer / SSL software (see the ideas
   mentioned above for the CA and server keys). The only difference is
   these certificates might be referred to as "My Certificates" or
   "Client Certificates". They are used for client authentication (which
   is relatively rare for SSL).

   After copying them you can delete the clients/*.pem files for extra
   safety because the private keys are not needed by the x11vnc server.
   You don't really need the clients/*.crt files either (because they
   have been signed by the CA). But they could come in handy for tracking
   or troubleshooting, etc.

   Now start up x11vnc and instruct it to verify connecting clients via
   SSL and the CA cert:
  x11vnc -ssl SAVE -sslverify CA

   The "CA" special token instructs x11vnc to use its CA signed certs for
   verification.

   For arbitrary self-signed client certificates (no CA) it might be
   something like this:
  x11vnc -ssl SAVE -sslverify path/to/client.crt
  x11vnc -ssl SAVE -sslverify path/to/client-hash-dir
  x11vnc -ssl SAVE -sslverify path/to/certs.txt

   Where client.crt would be an individual client certificate;
   client-hash-dir a directory of file names based on md5 hashes of the
   certs (see -sslverify); and certs.txt signifies a single file full of
   client certificates.

   Finally, connect with your VNC viewer using the key. Here is an
   example for the Unix stunnel wrapper script ss_vncviewer: using client
   authentication (and the standard server authentication with the CA
   cert):
  ss_vncviewer -mycert ./dilbert.pem -verify ./cacert.pem far-away.east:0

   Our SSVNC enhanced tightvnc viewer can also use these openssl .pem
   files (you can load them via Certs... -> MyCert dialog).

   It is also possible to use -sslverify on a per-client key basis, and
   also using self-signed client keys (x11vnc -sslGenCert client
   self:dilbert)

   Now a tricky part is to get Web browsers or Java Runtime to import and
   use the openssl .pem cert+key files. See the next paragraph on how to
   convert them to pkcs12 format. If you find a robust way to import them
   and and get them to use the cert please let us know!

   Here is how to convert our openssl crt/pem files to pkcs12 format
   (contains both the client certificate and key) that can be read by Web
   browsers and Java for use in client authentication:
  openssl pkcs12 -export -in mycert.crt -inkey mycert.pem -out mycert.p12

   it will ask for a passphrase to protect mycert.p12. Some software
   (e.g. Java ControlPanel) may require a non-empty passphrase. Actually,
   since our .pem contains both the certificate and private key, you
   could just supply it for the -in and remove the -inkey option. It
   appears that for certificates only importing, our .crt file is
   sufficient and can be read by Mozilla/Firefox and Java...

   If you have trouble getting your Java Runtime to import and use the
   cert+key, there is a workaround for the SSL-enabled Java applet. On
   the Web browser URL that retrieves the VNC applet, simply add a
   "/?oneTimeKey=..." applet parameter (see ssl-portal for more details
   on applet parameters; you don't need to do the full portal setup
   though). The value of the oneTimeKey will be the very long string that
   is output of the onetimekey program found in the classes/ssl x11vnc
   directory. Or you can set oneTimeKey=PROMPT in which case the applet
   will ask you to paste in the long string. These scheme is pretty ugly,
   but it works. A nice application of it is to make one time keys for
   users that have already logged into a secure HTTPS site via password.
   A cgi program then makes a one time key for the logged in user to use:
   it is passed back over HTTPS as the applet parameter in the URL and so
   cannot be sniffed. x11vnc is run to use that key via -sslverify.

   Update: as of Apr 2007 in the 0.9.1 x11vnc tarball there is a new
   option setting "-users sslpeer=" that will do a switch user much like
   -unixpw does, but this time using the emailAddress field of the
   Certificate subject of the verified Client. This mode requires
   -sslverify turned on to verify the clients via SSL. This mode can be
   useful in situations using -create or -svc where a new X server needs
   to be started up as the authenticated user (but unlike in -unixpw
   mode, the unix username is not obviously known).

     _________________________________________________________________

   Revoking Certificates:

   A large, scaled-up installation may benefit from being able to revoke
   certificates (e.g. suppose a user's laptop with a vnc client or server
   key is compromised.) You can use this option with x11vnc: -sslCRL. See
   the info at that link for a guide on what openssl(1) commands you will
   need to run to revoke a certificate.

     _________________________________________________________________

   Additional utlities:

   You can get information about your keys via -sslCertInfo. These lists
   all your keys:
  x11vnc -sslCertInfo list
  x11vnc -sslCertInfo ll

   (the latter is long format).

   These print long output, including the public certificate, for
   individual keys:
  x11vnc -sslCertInfo server
  x11vnc -sslCertInfo dilbert
  x11vnc -sslCertInfo all             (every key, very long)

   If you want to add a protecting passphrase to a key originally created
   without one:
  x11vnc -sslEncKey SAVE
  x11vnc -sslEncKey SAVE-fred

   To delete a cert+key:
  x11vnc -sslDelCert SAVE
  x11vnc -sslDelCert SAVE-fred
  x11vnc -sslDelCert wally

   (but rm(1) will be just as effective).

     _________________________________________________________________

   Chained Certificates:

   There is increasing interest in using chained CA's instead of a single
   CA. The merits of using chained CA's are not described here besides to
   say its use may make some things easier when a certificate needs to be
   revoked.

   x11vnc supports chained CA certificates. We describe a basic use case
   here.

   Background: Of course the most straight forward way to use SSL with
   x11vnc is to use no CA at all (see above): a self-signed certificate
   and key is used and its certificate needs to be safely copied to the
   client side. This is basically the same as the SSH style of managing
   keys. Next level up, one can use a single CA to sign server keys: then
   only the CA's certificate needs to be safely copied to the client
   side, this can happen even before any server certs are created (again,
   see all of the discussion above.)

   With a certificate chain there are two or more CA's involved. Perhaps
   it looks like this:
  root_CA ---> intermediate_CA ---> server_cert

   Where the arrow basically means "signs".

   In this usage mode the client (viewer-side) will have root_CA's
   certificate available for verifying (and nothing else.) If the viewer
   only received server_cert's certificate, it would not have enough info
   to verify the server. The client needs to have intermediate_CA's cert
   as well. The way to do this with x11vnc (i.e. an OpenSSL using app) is
   to concatenate the server_cert's pem and the intermediate_CA's
   certificate together.

   For example, suppose the file intermediate_CA.crt had
   intermediate_CA's certificate. And suppose the file server_cert.pem
   had the server's certificate and private key pair as described above
   on this page. We need to do this:
  cat intermediate_CA.crt >> server_cert.pem

   (Note: the order of the items inside the file matters; intermediate_CA
   must be after the server key and cert) and then we run x11vnc like
   this:
  x11vnc -ssl ./server_cert.pem ...

   Then, on the VNC viewer client side, the viewer authenticates the
   x11vnc server by using root_CA's certificate. Suppose that is in a
   file named root_CA.crt, then using the SSVNC wrapper script
   ss_vncviewer (which is also included in the SSVNC package) as our
   example, we have:
  ss_vncviewer -verify ./root_CA.crt hostname:0

   (where "hostname" is the machine where x11vnc is running.) One could
   also use the SSVNC GUI setting Certs -> ServerCert to the root_CA.crt
   file. Any other SSL enabled VNC viewer would use root_CA.crt in a
   similar way.
     _________________________________________________________________

   Creating Chained Certificates:

   Here is a fun example using VeriSign's "Trial Certificate" program.
   Note that VeriSign has a Root CA and also an Intermediate CA and uses
   the latter to sign customers certificates. So this provides an easy
   way to test out the chained certificates mechanism with x11vnc.

   First we created a test x11vnc server key:
  openssl genrsa -out V1.key 1024

   then we created a certificate signing request (CSR) for it:
  openssl req -new -key V1.key -out V1.csr

   (we followed the prompts and supplied information for the various
   fields.)

   Then we went to VeriSign's page http://www.verisign.com/ssl/index.html
   and clicked on "FREE TRIAL" (the certificate is good for 14 days.) We
   filled in the forms and got to the point where it asked for the CSR
   and so we pasted in the contents of the above V1.csr file. Then, after
   a few more steps, VeriSign signed and emailed us our certificate.

   The VeriSign Trial certificates were found here:
  http://www.verisign.com/support/verisign-intermediate-ca/Trial_Secure_Server_
Root/index.html
  http://www.verisign.com/support/verisign-intermediate-ca/trial-secure-server-
intermediate/index.html

   The former was pasted into a file V-Root.crt and the latter was pasted
   into V-Intermediate.crt

   We pasted our Trial certificate that VeriSign signed and emailed to us
   into a file named V1.crt and then we typed:
  cat V1.key V1.crt > V1.pem
  cat V1.pem V-Intermediate.crt > V1-combined.pem
  chmod 600 V1.pem V1-combined.pem

   So now the file V1-combined.pem has our private key and (VeriSign
   signed) certificate and VeriSign's Trial Intermediate certificate.

   Next, we start x11vnc:
  x11vnc -ssl ./V1-combined.pem ...

   and finally, on the viewer side (SSVNC wrapper script example):
  ss_vncviewer -verify ./V-Root.crt hostname:0

   One will find that only that combination of certs and keys will work,
   i.e. allow the SSL connection to be established. Every other
   combination we tried failed (note that ss_vncviewer uses the external
   stunnel command to handle the SSL so we are really testing stunnel's
   SSL implementation on the viewer side); and so the system works as
   expected.
     _________________________________________________________________

   VNC Client Authentication using Certificate Chains:

   Now, going the other way around with the client authenticating himself
   via this chain of SSL certificates, x11vnc is run this way:
  x11vnc -ssl SAVE -sslverify ./V-Root.crt ...

   (note since the server must always supply a cert, we use its normal
   self-signed, etc., one via "-ssl SAVE" and use the VeriSign root cert
   for client authentication via -sslverify. The viewer must now supply
   the combined certificates, e.g.:
  ss_vncviewer -mycert ./V1-combined.pem hostname:0
     _________________________________________________________________

   Using OpenSSL and x11vnc to create Certificate Chains:

   Although the x11vnc CA mechanism (-sslGenCA and -sslGenCert; see
   above) was designed to only handle a single root CA (to sign server
   and/or client certs) it can be coerced into creating a certificate
   chain by way of an extra openssl(1) command.

   We will first create two CA's via -sslGenCA; then use one of these CA
   to sign the other; create a new (non-CA) server cert; and append the
   intermediate CA's cert to the server cert to have everything needed in
   the one file.

   Here are the commands we ran to do what the previous paragraph
   outlines.

   First we create the two CA's, called CA_root and CA_Intermediate here,
   in separate directories via x11vnc:
  x11vnc -ssldir ~/CA_Root -sslGenCA
     (follow the prompts, we included "CA_Root", e.g. Common Name, to aid ident
ifying it)

  x11vnc -ssldir ~/CA_Intermediate -sslGenCA
     (follow the prompts, we included "CA_Intermediate", e.g. Common Name, to a
id identifying it)

   Next backup CA_Intermediate's cert and then sign it with CA_Root:
  mv ~/CA_Intermediate/CA/cacert.pem ~/CA_Intermediate/CA/cacert.pem.ORIG
  cd ~/CA_Root
  openssl ca -config ./CA/ssl.cnf -policy policy_anything -extensions v3_ca -no
text -ss_cert ~/CA_Intermediate/CA/cacert.pem.ORIG -out ~/CA_Intermediate/CA/ca
cert.pem

   Note that it is required to cd to the ~/CA_Root directory and run the
   openssl command from there.

   You can print out info about the cert you just modified by:
  openssl x509 -noout -text -in ~/CA_Intermediate/CA/cacert.pem

   Now we create an x11vnc server cert named "test_chain" that is signed
   by CA_Intermediate:
  x11vnc -ssldir ~/CA_Intermediate -sslGenCert server test_chain
     (follow the prompts)

   You can print out information about this server cert just created via
   this command:
  x11vnc -ssldir ~/CA_Intermediate -sslCertInfo SAVE-test_chain

   This will tell you the full path to the server certificate, which is
   needed because we need to manually append the CA_Intermediate cert for
   the chain to work:
  cat ~/CA_Intermediate/CA/cacert.pem >> ~/CA_Intermediate/server-test_chain.pe
m

   Now we are finally ready to use it. We can run x11vnc using this
   server cert+key by either this command:
  x11vnc -ssldir ~/CA_Intermediate -ssl SAVE-test_chain ...

   or this command:
  x11vnc -ssl ~/CA_Intermediate/server-test_chain.pem ...

   since they are equivalent (both load the same pem file.)

   Finally we connect via VNC viewer that uses CA_Root to verify the
   server. As before we use ss_vncviewer:
  ss_vncviewer -verify ~/CA_Root/CA/cacert.pem hostname:0

   Client Certificates (see above) work in a similar manner.

   So although it is a little awkward with the extra steps (e.g.
   appending the CA_Intermediate cert) it is possible. If you want to do
   this entirely with openssl(1) you will have to learn the openssl
   commands corresponding to -genCA and -genCert. You may be able to find
   guides on the Internet to do this. Starting with x11vnc 0.9.10, you
   can have it print out the wrapper scripts it uses via: -sslScripts
   (you will still need to fill in a few pieces of information; ask if it
   is not clear from the source code.)

     _________________________________________________________________

   More info:

   See also this article for some some general info and examples using
   stunnel and openssl on Windows with VNC. Also
   http://www.stunnel.org/faq/certs.html is a very good source of
   information on SSL certificate creation and management.
	
=======================================================================
http://www.karlrunge.com/x11vnc/ssl-portal.html:


     _________________________________________________________________

   Using Apache as an SSL Gateway to multiple x11vnc servers inside a
   firewall:

   Background:

   The typical way to allow access to x11vnc (or any other VNC server)
   running on multiple workstations inside a firewall is via SSH. The
   user somewhere out on the Internet logs in to the SSH gateway machine
   and uses port forwarding (e.g. ssh -t -L 5900:myworkstation:5900
   user@gateway) to set up the encrypted channel that VNC is then
   tunneled through. Next he starts up the VNC viewer on the machine
   where he is sitting directed to the local tunnel port (e.g.
   localhost:0).

   The SSH scheme is nice because it is a widely used and well tested
   login technique for users connecting to machines inside their company
   or home firewall. For VNC access it is a bit awkward, however, because
   SSH needs to be installed on the Viewer machine and the user usually
   has to rig up his own port redirection plumbing (however, see our
   other tool).

   Also, some users have restrictive work environments where SSH and
   similar applications are prohibited (i.e. only outgoing connections to
   standard WWW ports from a browser are allowed, perhaps mediated by a
   proxy server). These users have successfully used the method described
   here for remote access.

   With the SSL support in x11vnc and the SSL enabled Java VNC viewer
   applet, a convenient and secure alternative exists that uses the
   Apache webserver as a gateway. The idea is that the company or home
   internet connection is already running apache as a web server (either
   SSL or non-SSL) and we add to it the ability to act as a gateway for
   SSL VNC connections. The only thing needed on the Viewer side is a
   Java enabled Web Browser: the user simply enters a URL that starts the
   entire VNC connection process. No VNC or SSH specific software needs
   to be installed on the viewer side machine.

   The stunnel VNC viewer stunnel wrapper script provided (ss_vncviewer)
   can also take advantage of the method described here with its -proxy
   option.

     _________________________________________________________________

   Simpler Solutions: This apache SSL VNC portal solution may be too much
   for you. It is mainly intended for automatically redirecting to
   MULTIPLE workstations inside the firewall. If you only have one or two
   inside machines that you want to access, the method described here is
   overly complicated! See below for some simpler (and still non-SSH)
   encrypted setups.

   Also see the recent (Mar/2010) desktop.cgi x11vnc desktop web login
   CGI script that achieves much of what the method describes here
   (especially if its 'port redirection' feature is enabled.)
     _________________________________________________________________



   There are numerous ways to achieve this with Apache. We present one of
   the simplest ones here.

   Important: these sorts of schemes allow incoming connections from
   anywhere on the Internet to fixed ports on machines inside the
   firewall. Care must be taken to implement and test thoroughly. If one
   is paranoid one can (and should) add extra layers of protection. (e.g.
   extra passwords, packet filtering, SSL certificate verification, etc).

   Also, it is easy to miss the point that unless precautions are taken
   to verify SSL Certificates, then the VNC Viewer is vulnerable to
   man-in-the-middle attacks (but not to the more common passive sniffing
   attacks). Note that there are hacker tools like dsniff/webmitm and
   cain that implement SSL Man-In-The-Middle attacks. They rely on the
   client not bothering to check the cert.
     _________________________________________________________________

   The Holy Grail: a single https port (443)

   Before we discuss the self-contained apache examples here, we want to
   mention that many x11vnc users who read this page and implement the
   apache SSL VNC portal ask for something that (so far) seems difficult
   or impossible to do entirely inside apache:
     * A single port, 443 (the default https:// port), is open to the
       Internet
     * It is HTTPS/SSL encrypted
     * It handles both VNC traffic and Java VNC Applet downloads.
     * And the server can also serve normal HTTPS webpages, CGI, etc.

   It is the last item that makes it tricky (otherwise the method
   described on this page will work). If you are interested in such a
   solution and are willing to run a separate helper program
   (connect_switch) look here. Also, see this apache patch.
     _________________________________________________________________

   Example:

   The scheme described here sets up apache on the firewall/gateway as a
   regular Web proxy into the intranet and allows connections to a single
   fixed port on a limited set of machines.

   The configuration described in this section does not use the mod_ssl
   apache module (the optional configuration described in the section
   "Downloading the Java applet to the browser via HTTPS" does take
   advantage of mod_ssl)

   In this example suppose the gateway machine running apache is named
   "www.gateway.east" (e.g. it may also provide normal web service). We
   also choose the Internet-facing port for this VNC service to be port
   563. One could choose any port, including the default HTTP port 80.

   Detail: We choose 563 because it is the rarely used SNEWS port that is
   often allowed by Web proxies for the CONNECT method. The idea is the
   user may be coming out of another firewall using a proxy (not the one
   we describe here, that is, the case when two proxies are involved,
   e.g. one at work and another Apache (described here) at home
   redirecting into our firewall; the "double proxy" or "double firewall"
   problem). Using port 563 simplifies things because CONNECT's to it are
   usually allowed by default.

   We also assume all of the x11vnc servers on the internal machines are
   all listening on port 5915 ("-rfbport 5915") instead of the default
   5900. This is to limit any unintended proxy redirections to a lesser
   used port, and also to stay out of the way of normal VNC servers on
   the same machines. One could obviously implement a scheme that handles
   different ports, but we just discuss this simple setup here.

   So we basically assume x11vnc has been started this way on all of the
   workstations to be granted VNC access:
  x11vnc -ssl SAVE -http -display :0 -forever -rfbauth ~/.vnc/passwd -rfbport 5
915

   i.e. we force SSL VNC connections, port 5915, serve the Java VNC
   viewer applet, and require a VNC password (another option would be
   -unixpw). The above command could also be run out of inetd(8). It can
   also be used to autodetect the user's display and Xauthority data.


   These sections are added to the httpd.conf apache configuration file
   on www.gateway.east:

# In the global section you need to enable these modules.
# Note that the ORDER MATTERS! mod_rewrite must be before mod_proxy
# (so that we can check the allowed host list via rewrite)
#
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
LoadModule proxy_http_module modules/mod_proxy_http.so
<IfDefine SSL>
LoadModule ssl_module modules/mod_ssl.so
</IfDefine>


# Near the bottom of httpd.conf you put the port 563 virtual host:

Listen 563

<VirtualHost *:563>

   # Allow proxy CONNECT requests *only* to port 5915.
   # If the machines use different ports, e.g. 5916 list them here as well:
   #
   ProxyRequests On
   AllowCONNECT 5915

   RewriteEngine On

   # Convenience rules to expand applet parameters.  These do not have a traili
ng "/"
   #
   # /vnc   for http jar file downloading:
   #
   RewriteRule /vnc/([^/]+)$               /vnc/$1/index.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1 [R,NE,L]
   RewriteRule /vnc/trust/([^/]+)$         /vnc/$1/index.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&trustAllVncCerts=yes [R,NE,L]
   RewriteRule /vnc/proxy/([^/]+)$         /vnc/$1/proxy.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes [R,NE,L]
   RewriteRule /vnc/trust/proxy/([^/]+)$   /vnc/$1/proxy.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes&trustAllVncCerts=yes [R,NE,L]

   # Read in the allowed host to vnc display mapping file.  It looks like:
   #
   #   host1     15
   #   host2     15
   #   ...
   #
   # the display "15" means 5815 for http applet download, 5915 for SSL vnc.
   #
   RewriteMap vnchosts txt:/dist/apache/conf/vnc.hosts

   # Proxy: check for the CONNECT hostname and port being in the vnc.hosts list
.
   #
   RewriteCond %{THE_REQUEST} ^CONNECT [NC]
   RewriteCond %{REQUEST_URI} ^(.*):(.*)$
   RewriteCond ${vnchosts:%1|NOTFOUND} NOTFOUND
   RewriteRule ^.*$ /VNCFAIL [F,L]

   RewriteCond %{THE_REQUEST} ^CONNECT [NC]
   RewriteCond %{REQUEST_URI} ^(.*):(.*)$
   RewriteCond 59${vnchosts:%1}=%2 !^(.*)=(\1)$
   RewriteRule ^.*$ /VNCFAIL [F,L]


   # Remap /vnc to the proxy http download (e.g. http://host:5815)
   #
   # First, fail if it starts with the string /vnc0:
   #
   RewriteRule ^/vnc0.*            /VNCFAIL [F,L]
   #
   # Next, map the prefix to /vnc0/host:protocol:port
   #
   RewriteRule ^/vnc/([^/]+)/(.*)  /vnc0/$1:http:58${vnchosts:$1|NOTFOUND}/$2
[NE]
   #
   # Drop any not found:
   #
   RewriteRule ^/vnc0.*NOTFOUND.*  /VNCFAIL [F,L]

   # Construct the proxy URL and retrieve it:
   #
   RewriteRule ^/vnc0/([^/]+):([^/]+):([^/]+)/(.*) $2://$1:$3/$4 [P,NE,L]

</VirtualHost>

   Then restart apache (perhaps: "apachectl stop; apachectl start").

   Note that the listing of allowed internal workstations is done in an
   external file (/dist/apache/conf/vnc.hosts in the example above), the
   format is like this:
# allowed vnc hosts file:
hostname1  15
hostname2  15
...

   You list the hostname and the VNC display (always 15 in our example).
   Only to these hosts will the external VNC viewers be able to connect
   to (via the HTTP CONNECT method).

   The above setup requires mod_rewrite and mod_proxy be enabled in the
   apache web server. In this example they are loaded as modules (and
   note that mod_rewrite must be listed before mod_proxy);

   The user at the Java enabled Web browser would simply enter this URL
   into the browser:
   http://www.gateway.east:563/vnc/host2

   to connect to internal workstation host2, etc.

   Important: do not put a trailing "/" on the URL, since that will
   defeat the RewriteRules that look for the hostname at the very end.

   There will be a number of SSL certificate, etc, dialogs he will have
   to respond to in addition to any passwords he is required to provide
   (this depends on how you set up user authentication for x11vnc).

   If a second Web proxy is involved (i.e. the user's browser is inside
   another firewall and policy requires using a Web proxy server) then
   use this URL:
   http://www.gateway.east:563/vnc/proxy/host2

   This will involve downloading a signed java viewer applet jar file
   that is able to interact with the internal proxy for the VNC
   connection. See this FAQ for more info on how this works. Note:
   sometimes with the Proxy case if you see 'Bad Gateway' error you will
   have to wait 10 or so seconds and then hit reload. This seems to be
   due to having to wait for a Connection Keepalive to terminate...

   For completeness, the "trust" cases that skip a VNC certificate dialog
   (discussed below) would be entered as:
   http://www.gateway.east:563/vnc/trust/host2
   http://www.gateway.east:563/vnc/trust/proxy/host2

   You can of course choose shorter or more easy to remember URL formats.
   Just change the Convenience RewriteRules in httpd.conf.

     _________________________________________________________________

   Port Variations:

   Note that you can run this on the default HTTP port 80 instead of port
   563. If you do not expect to have a browser connecting from inside a
   proxying firewall (where sometimes only connections to ports 443 and
   563 are allowed) this should be fine. Use "80" instead of "563" in the
   httpd.conf config file (you may need to merge it with other default
   port 80 things you have there).

   Then the URL's will be a bit simpler:
   http://www.gateway.east/vnc/host2
   http://www.gateway.east/vnc/trust/host2

   etc.

   Besides 80 one could use any other random port number (since there are
   so many port scans on 80, a little obscurity might be useful).

   One option is to use port "443" (the default https:// port) instead of
   "563". In this case Apache is not configured for mod_ssl; we just
   happen to use port "443" in the way any random port would be used.
   This could be handy if the Viewer side environment is restrictive in
   that it only allows outgoing connections to ports 80 and 443 (and,
   say, you didn't want to use port 80, or you wanted to use 80 for
   something else). Another reason for using 443 would be some web proxy
   environments only allow the CONNECT method to go to port 443 (and not
   even the case 563 we use above).

     _________________________________________________________________

   Details:

   Let's go through the httpd.conf additions in detail from the top.

   The LoadModules directives load the necessary apache modules. Note
   that mod_rewrite must be listed first. If you are compiling from
   scratch something like this worked for us:
  ./configure --enable-proxy=shared --enable-proxy-connect=shared --enable-ssl=
shared --enable-rewrite=shared --prefix=/dist/apache

   Then the VirtualHost *:563 virtual host section starts.

   The "ProxyRequests On" and "AllowCONNECT 5915" enable the web server
   to forward proxy requests to port 5915 (and only this port) INSIDE the
   firewall. Think about the implications of this thoroughly and test it
   carefully.

   The RewriteRule's are for convenience only so that the URL entered
   into the Web browser does not need the various extra parameters, e.g.:
   http://www.gateway.east:563/vnc/host2/index.vnc?CONNECT=host2+5915&PORT=563,
blah,blah...

   (or otherwise make direct edits to index.vnc to set these parameters).
   The forceProxy=yes parameter is passed to the applet to force the use
   of a outgoing proxy socket connection. Use it only if the Web browser
   is inside a separate Web proxying environment (i.e. large corporation)

   The rewrites with parameter urlPrefix are described under Tricks for
   Better Response. The "trust" ones (also described under Tricks) with
   trustAllVncCerts tell the Java VNC applet to skip a dialog asking
   about the VNC Certificate. They are a bit faster and more reliable
   than the original method. In the best situation they lead to being
   logged in 20 seconds or less (without them the time to login can be
   much longer since a number of connections must timeout).

   All of the x11vnc Java Viewer applet parameters are described in the
   file classes/ssl/README

   The external file /dist/apache/conf/vnc.hosts containing the allowed
   VNC server hostnames is read in. Its 2nd column contains the VNC
   display of the host (always 15 in our example; if you make it vary you
   will need to adjust some lines in the httpd.conf accordingly, e.g.
   AllowCONNECT). This list is used to constrain both the Jar file
   download URL and the proxy CONNECT the VNC viewer makes to only the
   intended VNC servers.

   Limiting the proxy CONNECT is done with the two sets of RewriteCond
   conditions.

   Limiting the Jar file download URL is done in the remaining 4
   RewriteRule's.

   Note that these index.vnc and VncViewer.jar downloads to the browser
   are not encrypted via SSL, and so in principle could be tampered with
   by a really bad guy. The subsequent VNC connection, however, is
   encrypted through a single SSL connection (it makes a CONNECT straight
   to x11vnc). See below for how to have these initial downloads
   encrypted as well (if the apache web server has SSL/mod_ssl, i.e.
   https, enabled and configured).

   Unfortunately the Java VNC viewer applet currently is not able to save
   its own list of Certificates (e.g. the user says trust this VNC
   certificate 'always'). This is because an applet it cannot open local
   files, etc. Sadly, the applet cannot even remember certificates in the
   same browser session because it is completely reinitialized for each
   connection (see below).

     _________________________________________________________________

   Too Much?

   If these apache rules are a little too much for you, there is a little
   bit simpler scheme where you have to list each of the individual
   machines in the httpd.conf and ssl.conf files. It may be a little more
   typing to maintain, but perhaps being more straight forward (less
   RewriteRule's) is desirable.

     _________________________________________________________________

   Problems?

   To see example x11vnc output for a successful https://host:5900/
   connection with the Java Applet see This Page.

     _________________________________________________________________

   Some Ideas for adding extra authentication, etc. for the paranoid:
     * VNC passwords: -rfbauth, -passwdfile, or -usepw. Even adding a
       simple company-wide VNC password helps block unwanted access.
     * Unix passwords: -unixpw
     * SSL Client certificates: -sslverify
     * Apache AuthUserFile directive: .htaccess, etc.
     * Filter connections based on IP address or hostname.
     * Use Port-knocking on your firewall as described in: Enhanced
       TightVNC Viewer (ssvnc).
     * Add proxy password authentication (requires Viewer changes?)
     * Run a separate instance of Apache that provides this VNC service
       so it can be brought up and down independently of the normal web
       server.
     * How secure is the Client side? Public machines in internet cafes,
       etc, are often hacked, with backdoors and VNC servers of their
       own. Prefer using your own firewalled laptop to a public machine.


     _________________________________________________________________

   Using non-Java viewers with this scheme:

   The ss_vncviewer stunnel wrapper script for VNC viewers has the -proxy
   option that can take advantage of this method.
   ss_vncviewer -proxy www.gateway.east:563   host1:15

   For the case of the "double proxy" situation (see below) supply both
   separated by a comma.
   ss_vncviewer -proxy proxy1.foobar.com:8080,www.gateway.east:563   host1:15

   For the Enhanced TightVNC Viewer (ssvnc) GUI (it uses ss_vncviewer on
   Unix) put 'host1:15' into the 'VNC Server' entry box, and here are
   possible Proxy/Gateway entries
   Proxy/Gateway:   www.gateway.east:563
   Proxy/Gateway:   proxy1.foobar.com:8080,www.gateway.east:563

   then click on the 'Connect' button.

     _________________________________________________________________

   Downloading the Java applet to the browser via HTTPS:

   To have the Java applet downloaded to the user's Web Browser via an
   encrypted (and evidently safer) SSL connection the Apache webserver
   should be configured for SSL via mod_ssl.

   It is actually possible to use the x11vnc Key Management utility
   "-sslGenCert" to generate your Apache/SSL .crt and .key files. (In
   brief, run something like "x11vnc -sslGenCert server self:apache" then
   copy the resulting self:apache.crt file to conf/ssl.crt/server.crt and
   extract the private key part from self:apache.pem and paste it into
   conf/ssl.key/server.key). Setting the env var REQ_ARGS='-days 1095'
   before running x11vnc will bump up the expiration date (3 years in
   this case).

   Or you can use the standard methods described in the Apache mod_ssl
   documentation to create your keys. Then restart Apache, usually
   something like "apachectl stop" followed by "apachectl startssl"

   In addition to the above sections in httpd.conf one should add the
   following to ssl.conf:
   SSLProxyEngine  On

   RewriteEngine On

   # Convenience rules to expand applet parameters.  These do not have a traili
ng "/"
   #
   # /vnc   http jar file downloading:
   #
   RewriteRule /vnc/([^/]+)$                        /vnc/$1/index.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vnc_2F_$1 [R,NE,L]
   RewriteRule /vnc/proxy/([^/]+)$                  /vnc/$1/proxy.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes [R,N
E,L]
   #
   # (we skipped the "trust" ones above, put them in if you like)
   #
   # /vncs  https jar file downloading:
   #
   RewriteRule /vncs/([^/]+)$                      /vncs/$1/index.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1 [R,NE,L]
   RewriteRule /vncs/proxy/([^/]+)$                /vncs/$1/proxy.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes [R,
NE,l]
   RewriteRule /vncs/trust/([^/]+)$                /vncs/$1/index.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&trustAllVncCerts=y
es [R,NE,L]
   RewriteRule /vncs/trust/proxy/([^/]+)$          /vncs/$1/proxy.vnc?CONNECT=$
1+5915&PORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes&tru
stAllVncCerts=yes [R,NE,L]

   # Convenience rules used for the connect_switch helper (requires Listen 127.
0.0.1:443 above):
   #
   RewriteRule /vnc443/([^/]+)$                    /vncs/$1/index.vnc?CONNECT=$
1+5915&PORT=443&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1 [R,NE,L]
   RewriteRule /vnc443/proxy/([^/]+)$              /vncs/$1/proxy.vnc?CONNECT=$
1+5915&PORT=443&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes [R,
NE,L]
   RewriteRule /vnc443/trust/([^/]+)$              /vncs/$1/index.vnc?CONNECT=$
1+5915&PORT=443&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&trustAllVncCerts=y
es [R,NE,L]
   RewriteRule /vnc443/trust/proxy/([^/]+)$        /vncs/$1/proxy.vnc?CONNECT=$
1+5915&PORT=443&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes&tru
stAllVncCerts=yes [R,NE,L]

   # Read in the allowed host to vnc display mapping file.  It looks like:
   #
   #   host1     15
   #   host2     15
   #   ...
   #
   # the display "15" means 5915 for SSL VNC and 5815 for http applet download.
   #
   RewriteMap vnchosts txt:/dist/apache/conf/vnc.hosts


   # Remap /vnc and /vncs to the proxy http download (e.g. https://host:5915)
   #
   # First, fail if it starts with the string /vnc0:
   #
   RewriteRule ^/vnc0.*            /VNCFAIL [F,L]
   #
   # Next, map the prefix to /vnc0:host:protocol:port
   #
   RewriteRule ^/vnc/([^/]+)/(.*)  /vnc0/$1:http:58${vnchosts:$1|NOTFOUND}/$2
[NE]
   RewriteRule ^/vncs/([^/]+)/(.*) /vnc0/$1:https:59${vnchosts:$1|NOTFOUND}/$2
[NE]
   #
   # Drop any not found:
   #
   RewriteRule ^/vnc0.*NOTFOUND.*  /VNCFAIL [F,L]

   # Construct the proxy URL and retrieve it:
   #
   RewriteRule ^/vnc0/([^/]+):([^/]+):([^/]+)/(.*) $2://$1:$3/$4 [P,NE,L]

   This is all in the "<VirtualHost _default_:443>" section of ssl.conf.

   The user could then point the Web Browser to:
   https://www.gateway.east/vnc/host2

   or
   https://www.gateway.east/vnc/proxy/host2

   for the "double proxy" case. (Important: do not put a trailing "/" on
   the URL, since that will defeat the RewriteRules.)

   As with the httpd.conf case, the external file
   (/dist/apache/conf/vnc.hosts in the above example) contains the
   hostnames of the allowed VNC servers.

   Note that inside the firewall the Java applet download traffic is not
   encrypted (only over the Internet is SSL used) for these cases:
   https://www.gateway.east/vnc/host2
   https://www.gateway.east/vnc/proxy/host2

   However for the special "vncs" rules above:
   https://www.gateway.east/vncs/host2

   the Java applet download is encrypted via SSL for both legs. Note that
   the two legs are two separate SSL sessions. So the data is decrypted
   inside an apache process and reencrypted by the apache process for the
   2nd SSL session inside the same apache process (a very small gap one
   might overlook).

   The "vncs/trust" ones are like the "trust" ones described earlier
   https://www.gateway.east/vncs/trust/mach2

   and similarly for the httpsPort ones. See Tricks for Better Response.

   In all of the above cases the VNC traffic from Viewer to x11vnc is
   encrypted end-to-end in a single SSL session, even for the "double
   proxy" case because the CONNECT method is used (there are actually two
   CONNECT's for the "double proxy" case). This part (the VNC traffic) is
   the most important part to have encrypted.

   Note that the Certificate dialogs the user has in his web browser will
   be for the Apache Certificate, while for the Java applet it will be
   the x11vnc certificate.

   Note also that you can have Apache serve up the Jar file VncViewer.jar
   and/or index.vnc/proxy.vnc instead of each x11vnc if you want to.

   The rules in ssl.conf are similar to the ones in httpd.conf and so are
   not discussed in detail. The only really new thing is the /vncs
   handling to download the applet jar via HTTPS on port 5915.

   The special entries "/vnc443" are only used for the special helper
   program (connect_switch) for the https port 443 only mode discussed
   here.

     _________________________________________________________________

   INETD automation:

   The "single-port" (i.e. 5915) HTTPS applet download and VNC connection
   aspect shown here is convenient and also enables having x11vnc run out
   of inetd. That way x11vnc is run on demand instead of being run all
   the time (the user does not have to remember to start it). The first
   connections to inetd download index.vnc and the Jar file (via https)
   and the the last connection to inetd establishes the SSL VNC
   connection. Since x11vnc is restarted for each connection, this will
   be a bit slower than the normal process.

   For example, the /etc/inetd.conf line could be:
  5915 stream tcp nowait root /usr/sbin/tcpd /usr/local/bin/x11vnc_ssl.sh

   where the script x11vnc_ssl.sh looks something like this:
#!/bin/sh

/usr/local/bin/x11vnc -inetd -oa /var/log/x11vnc-15.log \
        -ssl SAVE -http -unixpw -localhost \
        -display :0 -auth /home/THE_USER/.Xauthority

   where, as usual, the inetd launching needs to know which user is
   typically using the display on that machine. One could imagine giving
   different users different ports, 5915, 5916, etc. to distinguish (then
   the script would need to be passed the username). mod_rewrite could be
   used to automatically map username in the URL to his port number.

   A better way is to use the "-display WAIT:cmd=FINDDISPLAY" feature to
   autodetect the user and Xauthority data:
#!/bin/sh

/usr/local/bin/x11vnc -inetd -oa /var/log/x11vnc-15.log \
        -ssl SAVE -http -unixpw -localhost -users unixpw= \
        -find

   (we have used the alias -find for "-display WAIT:cmd=FINDDISPLAY".)
   This way the user must supply his Unix username and password and then
   his display and Xauthority data on that machine will be located and
   returned to x11vnc to allow it to attach. If he doesn't have a display
   running on that machine or he fails to log in correctly, the
   connection will be dropped.

   The variant "-display WAIT:cmd=FINDCREATEDISPLAY" (aliased by
   "-create") will actually create a (virtual or real) X server session
   for the user if one doesn't already exist. See here for details.

   To enable inetd operation for the non-HTTPS Java viewer download (port
   5815 in the above httpd.conf example) you will need to run x11vnc in
   HTTPONCE mode on port 5815: For example, the /etc/inetd.conf line
   could be:
  5815 stream tcp nowait root /usr/sbin/tcpd /usr/local/bin/x11vnc \
       -inetd -prog /usr/local/bin/x11vnc -oa /var/log/x11vnc-15.log \
       -http_ssl -display WAIT:cmd=HTTPONCE

   where the long inetd.conf line has been split. Note how the -http_ssl
   tries to automatically find the .../classes/ssl subdirectory. This
   requires the -prog option available in x11vnc 0.8.4 (a shell script
   wrapper, e.g. /usr/local/bin/x11vnc_http.sh can be used to work around
   this).

   Also note the use of "-ssl SAVE" above. This way a saved server.pem is
   used for each inetd invocation (rather generating a new one each time
   as happens for "-ssl TMP"). Note that it cannot have a protecting
   passphrase because inetd will not be able to supply it.

   Another option is:
  5815 stream tcp nowait root /usr/sbin/tcpd /usr/local/bin/x11vnc \
       -inetd -httpdir /usr/local/share/x11vnc/classes/ssl \
       -oa /var/log/x11vnc-15.log -display WAIT:cmd=HTTPONCE

   (this also requires a feature found in x11vnc 0.8.4).
     _________________________________________________________________

   Other Ideas:

   - The above schemes work, but they are a bit complicated with all of
   the rigging. There should be more elegant ways to configure Apache to
   do these, but we have not found them (please let us know if you
   discover something nice). However, once this scheme has been set up
   and is working it is easy to maintain and add/delete workstations,
   etc.

   - In general Apache is not required, but it makes things convenient.
   The firewall itself could do the port redirection via its firewall
   rules. Evidently different Internet-facing ports would be required for
   each workstation. This could be set up using iptables rules for
   example. If there were just one or two machines this would be the
   easiest method. For example:
  iptables -t nat -A PREROUTING -p tcp -d 24.35.46.57 --dport 5901 -j DNAT --to
-destination 192.168.1.2:5915
  iptables -t nat -A PREROUTING -p tcp -d 24.35.46.57 --dport 5902 -j DNAT --to
-destination 192.168.1.3:5915

   Where 24.35.46.57 is the internet IP address of the gateway. In this
   example 24.35.46.57:5901 is redirected to the internal machine
   192.168.1.2:5915 and 24.35.46.57:5902 is redirected to another
   internal machine 192.168.1.3:5915, both running x11vnc -ssl ... in SSL
   mode. For this example, the user would point the web browser to, e.g.:
  https://24.35.46.57:5901/?PORT=5901

   or using the stunnel wrapper script:
  ss_vncviewer 24.35.46.57:1

   One can achieve similar things with dedicated firewall/routers (e.g.
   Linksys) using the device's web or other interface to configure the
   firewall.

   If the user may be coming out of a firewall using a proxy it may be
   better to redirect ports 443 and 563 (instead of 5901 and 5902) to the
   internal machines so that the user's proxy will allow CONNECTing to
   them.

   - The redirection could also be done at the application level using a
   TCP redirect program (e.g. ip_relay or fancier ones). Evidently more
   careful internal hostname checking, etc., could be performed by the
   special purpose application to add security. See connect_switch which
   is somewhat related.

   - One might imagine the ProxyPass could be done for the VNC traffic as
   well (for the ssl.conf case) to avoid the CONNECT proxying completely
   (which would be nice to avoid). Unfortunately we were not able to get
   this to work. Since HTTP is a request-response protocol (as opposed to
   a full bidirectional link required by VNC that CONNECT provides) this
   makes it difficult to do. It may be possible, but we haven't found out
   how yet.

   All of the x11vnc Java Viewer applet parameters are described in the
   file classes/ssl/README

     _________________________________________________________________

   Tricks for Better Response and reliability:

   The "original scheme" using httpd.conf and ssl.conf rewrites without
   urlPrefix and trustAllVncCerts above should work OK, but may lead to
   slow and/or unreliable loading of the applet and final connection to
   x11vnc. The following are what I do now to get better response and
   reliability. YMMV.

   The problem with the "original scheme" is that there is a point where
   the VNC Viewer applet can try up to 3 times to retrieve the x11vnc
   certificate, since it needs to get it to show it to you and ask you if
   you accept it. This can add about 45 seconds to the whole process
   (which takes 1 to 1.5 minutes with all the dialogs) since a couple of
   those connections must time out. The "trust" items in the config add a
   parameter trustAllVncCerts=yes similar to the forceProxy=yes
   parameter. This can cut the total time to the VNC password prompt down
   to 15 seconds which is pretty good. (Note by ignoring the certificate
   this does not protect against man-in-the-middle attacks which are
   rare, but maybe the won't be so rare in the future... see
   dsniff/webmitm and cain)

   First make sure the x11vnc SSL certificate+key is the same as
   Apache's. (otherwise you may get one extra dialog and/or one extra
   connection that has to time out).

   The following RewriteRule's are the same now advocated in the
   instructions above.

   The httpsPort and urlPrefix= parameters give hints to the applet to
   improve connecting: This is what goes in httpd.conf:
   RewriteEngine On
   RewriteRule /vnc/([^/]+)$               /vnc/$1/index.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1 [R,NE]
   RewriteRule /vnc/trust/([^/]+)$         /vnc/$1/index.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&trustAllVncCerts=yes [R,NE]
   RewriteRule /vnc/proxy/([^/]+)$         /vnc/$1/proxy.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes [R,NE]
   RewriteRule /vnc/trust/proxy/([^/]+)$   /vnc/$1/proxy.vnc?CONNECT=$1+5915&PO
RT=563&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes&trustAllVncCerts=yes [R,NE]

   The httpsPort and urlPrefix provide useful hints to the VNC Viewer
   applet when it connects to x11vnc to glean information about Proxies,
   certificates, etc.

   This is what goes into ssl.conf:
   RewriteEngine On
   RewriteRule /vnc/([^/]+)$                /vnc/$1/index.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vnc_2F_$1 [R,NE]
   RewriteRule /vnc/proxy/([^/]+)$          /vnc/$1/proxy.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vnc_2F_$1&forceProxy=yes [R,NE]
   RewriteRule /vncs/([^/]+)$              /vncs/$1/index.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1 [R,NE]
   RewriteRule /vncs/proxy/([^/]+)$        /vncs/$1/proxy.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes [R,NE]
   RewriteRule /vncs/trust/([^/]+)$        /vncs/$1/index.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&trustAllVncCerts=yes [R,NE
]
   RewriteRule /vncs/trust/proxy/([^/]+)$  /vncs/$1/proxy.vnc?CONNECT=$1+5915&P
ORT=563&httpsPort=443&GET=1&urlPrefix=_2F_vncs_2F_$1&forceProxy=yes&trustAllVnc
Certs=yes [R,NE]

   The rest is the same.

   The httpsPort and urlPrefix and GET provide useful hints to the VNC
   Viewer applet when it connects to x11vnc to glean information about
   Proxies, certificates, etc, and also for the ultimate VNC connection
   (GET speeds this up by sending a special HTTP GET to cause x11vnc to
   immediately switch to the VNC protocol).

   To turn these into URLs, as was done above, take the string in the
   RewriteRule, e.g. /vncs and turn it into
   https://gateway/vncs/machinename Similarly for non-https:
   http://gateway:563/vnc/machinename

   If you use the 'trust' ones, you are performing NO checks, visual or
   otherwise, on the VNC SSL certificate. It is trusted without question.
   This speeds things up because it avoids a dialog about certificates,
   but of course has some risk WRT Man in the Middle attacks. I don't
   recommend them. It is better to use /vnc or /vncs and the first time
   you connect carefully check the Certificate and then tell your Browser
   and Java Virtual Machine to trust the certificate 'Always'. Then if
   you later get an unexpected dialog, you know something is wrong.
   Nearly always it is just a changed or expired certificate, but better
   safe than sorry...
	
=======================================================================
http://www.karlrunge.com/x11vnc/enhanced_tightvnc_viewer.html:


     _________________________________________________________________

Enhanced TightVNC Viewer   (SSVNC:   SSL/SSH VNC viewer)

   (To Downloads)  (To Quick Start)

   [ssvnc.gif] [ssvnc_windows.gif] [ssvnc_macosx.gif] . .


   The Enhanced TightVNC Viewer, SSVNC, adds encryption security to VNC
   connections.

   The package provides a GUI for Windows, Mac OS X, and Unix that
   automatically starts up an STUNNEL SSL tunnel for SSL or ssh/plink for
   SSH connections to any VNC server, such as x11vnc, and then launches
   the VNC Viewer to use the encrypted tunnel.

   The x11vnc server has built-in SSL support, however SSVNC can make SSL
   encrypted VNC connections to any VNC Server if they are running an SSL
   tunnel, such as STUNNEL or socat, at their end. SSVNC's SSH tunnel
   will work to any VNC Server host running sshd that you can log into.

   The Enhanced TightVNC Viewer package started as a project to add some
   patches to the long neglected Unix TightVNC Viewer. However, now the
   front-end GUI, encryption, and wrapper scripts features possibly
   outweigh the Unix TightVNC Viewer improvements (see the lists below to
   compare).

   The SSVNC Unix vncviewer can also be run without the SSVNC encryption
   GUI as an enhanced replacement for the xvncviewer, xtightvncviewer,
   etc., viewers.

   In addition to normal SSL, SSVNC also supports the VeNCrypt SSL/TLS
   and Vino/ANONTLS encryption extensions to VNC on Unix, Mac OS X, and
   Windows. Via the provided SSVNC VeNCrypt bridge, VeNCrypt and ANONTLS
   encryption also works with any third party VNC Viewer (e.g. RealVNC,
   TightVNC, UltraVNC, etc...) you select via 'Change VNC Viewer'.

   The short name for this project is "ssvnc" for SSL/SSH VNC Viewer.
   This is the name of the command to start it.

   There is a simplified SSH-Only mode (sshvnc). And an even more
   simplified Terminal-Services mode (tsvnc) for use with x11vnc on the
   remote side.

   The tool has many additional features; see the descriptions below.

   It is a self-contained bundle, you could carry it around on, say, a
   USB memory stick / flash drive for secure VNC viewing from almost any
   machine, Unix, Mac OS X, and Windows (and if you create a directory
   named "Home" in the toplevel ssvnc directory on the drive your VNC
   profiles and certs will be kept there as well). For Unix, there is
   also a conventional source tarball to build and install in the normal
   way and not use a pre-built bundle.

     _________________________________________________________________

    Announcements:

   Important: If you created any SSL certificates with SSVNC (or anything
   else) on a Debian or Ubuntu system from Sept. 2006 through May 2008,
   then those keys are likely extremely weak and can be easily cracked.
   The certificate files should be deleted and recreated on a non-Debian
   system or an updated one. See
   http://www.debian.org/security/2008/dsa-1571 for details. The same
   applies to SSH keys.

   Please read this information on using SSVNC on workstations with
   Untrusted Local Users.

     _________________________________________________________________

    Feature List:

   Wrapper scripts and a tcl/tk GUI were written to create these features
   for Unix, Mac OS X, and Windows:
     * SSL support for connections using the bundled stunnel program.
     * Automatic SSH connections from the GUI (system ssh is used on Unix
       and MacOS X; bundled plink is used on Windows)
     * Ability to Save and Load VNC profiles for different hosts.
     * You can also use your own VNC Viewer, e.g. UltraVNC or RealVNC,
       with the SSVNC encryption GUI front-end if you prefer.
     * Create or Import SSL Certificates and Private Keys.
     * Reverse (viewer listening) VNC connections via SSL and SSH.
     * VeNCrypt SSL/TLS VNC encryption support (used by VeNCrypt, QEMU,
       ggi, libvirt/virt-manager/xen, vinagre/gvncviewer/gtk-vnc)
     * ANONTLS SSL/TLS VNC encryption support (used by Vino)
     * VeNCrypt and ANONTLS are also enabled for any 3rd party VNC Viewer
       (e.g. RealVNC, TightVNC, UltraVNC ...) on Unix, MacOSX, and
       Windows via the provided SSVNC VeNCrypt Viewer Bridge tool (use
       'Change VNC Viewer' to select the one you want.)
     * Support for Web Proxies, SOCKS Proxies, and the UltraVNC repeater
       proxy (e.g. repeater://host:port+ID:1234). Multiple proxies may be
       chained together (3 max).
     * Support for SSH Gateway connections and non-standard SSH ports.
     * Automatic Service tunnelling via SSH for CUPS and SMB Printing,
       ESD/ARTSD Audio, and SMB (Windows/Samba) filesystem mounting.
     * Sets up any additional SSH port redirections that you want.
     * Zeroconf (aka Bonjour) is used on Unix and Mac OS X to find VNC
       servers on your local network if the avahi-browse or dns-sd
       program is available and in your PATH.
     * Port Knocking for "closed port" SSH/SSL connections. In addition
       to a simple fixed port sequence and one-time-pad implementation, a
       hook is also provided to run any port knocking client before
       connecting.
     * Support for native MacOS X usage with bundled Chicken of the VNC
       viewer (the Unix X11 viewer is also provided for MacOS X, and is
       better IMHO. It is now the default on MacOS X.)
     * Dynamic VNC Server Port determination and redirection (using ssh's
       builtin SOCKS proxy, ssh -D) for servers like x11vnc that print
       out PORT= at startup.
     * Unix Username and Password entry for use with "x11vnc -unixpw"
       type login dialogs.
     * Simplified mode launched by command "sshvnc" that is SSH Only.
     * Simplified mode launched by command "tsvnc" that provides a VNC
       "Terminal Services" mode (uses x11vnc on the remote side).
     * IPv6 support for all connection modes on Unix, MacOSX, and
       Windows.

   Patches to TightVNC 1.3.9 vnc_unixsrc tree were created for Unix
   TightVNC Viewer improvements (these only apply to the Unix VNC viewer,
   including MacOSX XQuartz):
     * rfbNewFBSize VNC support (dynamic screen resizing)
     * Client-side Scaling of the Desktop in the viewer.
     * ZRLE VNC encoding support (RealVNC's encoding)
     * Support for the ZYWRLE encoding, a wavelet based extension to ZRLE
       to improve compression of motion video and photo regions.
     * TurboVNC support (VirtualGL's modified TightVNC encoding; requires
       TurboJPEG library)
     * Pipelined Updates of the framebuffer as in TurboVNC (asks for the
       next update before the current one has finished downloading; this
       gives some speedup on high latency connections.)
     * Cursor alphablending with x11vnc at 32bpp (-alpha option)
     * Option "-unixpw ..." for use with "x11vnc -unixpw" type login
       dialogs.
     * Support for UltraVNC extensions: 1/n Server side scaling, Text
       Chat, Single Window, Disable Server-side Input. Both UltraVNC and
       x11vnc servers support these extensions.
     * UltraVNC File Transfer via an auxiliary Java helper program (java
       must be in $PATH). Note that the x11vnc server also supports
       UltraVNC file transfer.
     * Connection support for the UltraVNC repeater proxy (-repeater
       option).
     * Support for UltraVNC Single Click operation. (both unencrypted: SC
       I, and SSL encrypted: SC III)
     * Support for UltraVNC DSM Encryption Plugin symmetric encryption
       mode. (ARC4, AESV2, MSRC4, and SecureVNC)
     * Support for UltraVNC MS-Logon authentication (NOTE: the UltraVNC
       MS-Logon key exchange implementation is very weak; an eavesdropper
       on the network can recover your Windows password easily in a few
       seconds; you need to use an additional encrypted tunnel with
       MS-Logon.)
     * Support for symmetric encryption (including blowfish and 3des
       ciphers) to Non-UltraVNC Servers. Any server using the same
       encryption method will work, e.g.:  x11vnc -enc blowfish:./my.key
     * Instead of hostname:display one can also supply "exec=command
       args..." to connect the viewer to the stdio of an external command
       (e.g. stunnel or socat) rather than using a TCP/IP socket. Unix
       domain sockets, e.g. /path/to/unix/socket, and a previously opened
       file descriptor fd=0, work too.
     * Local Port Protections for STUNNEL and SSH: avoid having for long
       periods of time a listening port on the the local (VNC viewer)
       side that redirects to the remote side.
     * Reverse (viewer listening) VNC connections can show a Popup dialog
       asking whether to accept the connection or not (-acceptpopup.) The
       extra info provided by UltraVNC Single Click reverse connections
       is also supported (-acceptpopupsc)
     * Extremely low color modes: 64 and 8 colors in 8bpp
       (-use64/-bgr222, -use8/-bgr111)
     * Medium color mode: 16bpp mode on a 32bpp Viewer display
       (-16bpp/-bgr565)
     * For use with x11vnc's client-side caching -ncache method use the
       cropping option -ycrop n. This will "hide" the large pixel buffer
       cache below the actual display. Set to the actual height or use -1
       for autodetection (also, tall screens, H > 2*W, are autodetected
       by default).
     * Escape Keys: specify a set of modifier keys so that when they are
       all pressed down you can invoke Popup menu actions via keystrokes.
       I.e., a set of 'Hot Keys'. One can also pan (move) the desktop
       inside the viewport via Arrow keys or a mouse drag.
     * Scrollbar width setting: -sbwidth n, the default is very thin, 2
       pixels, for less distracting -ycrop usage.
     * Selection text sending and receiving can be fine-tuned with the
       -sendclipboard, -sendalways, and -recvtext options.
     * TightVNC compression and quality levels are automatically set
       based on observed network latency (n.b. not bandwidth.)
     * Improvements to the Popup menu, all of these can now be changed
       dynamically via the menu: ViewOnly, Toggle Bell, CursorShape
       updates, X11 Cursor, Cursor Alphablending, Toggle Tight/ZRLE,
       Toggle JPEG, FullColor/16bpp/8bpp (256/64/8 colors), Greyscale for
       low color modes, Scaling the Viewer resolution, Escape Keys,
       Pipeline Updates, and others, including UltraVNC extensions.
     * Maintains its own BackingStore if the X server does not.
     * The default for localhost:0 connections is not raw encoding since
       same-machine connections are pretty rare. Default assumes you are
       using a SSL or SSH tunnel. Use -rawlocal to revert.
     * XGrabServer support for fullscreen mode, for old window managers
       (-grab/-graball option).
     * Fix for Popup menu positioning for old window managers (-popupfix
       option).
     * The VNC Viewer ssvncviewer supports IPv6 natively (no helpers
       needed.)

   The list of 3rd party software bundled in the archive files:
     * TightVNC Viewer  (windows, unix, macosx)
     * Chicken of the VNC Viewer  (macosx)
     * Stunnel  (windows, unix, macosx)
     * Putty/Plink/Pageant  (windows)
     * OpenSSL  (windows)
     * esound  (windows)

   These are all self-contained in the bundle directory: they will not be
   installed on your system. Just un-zip or un-tar the file you
   downloaded and run the frontend ssvnc straight from its directory.
   Alternatively, on Unix you can use the conventional source tarball.

     _________________________________________________________________

   Here is the Quick Start info from the README for how to setup and use
   SSVNC:
Quick Start:
-----------

Unix and Mac OS X:

    Inside a Terminal do something like the following.

    Unpack the archive:

        % gzip -dc ssvnc-1.0.29.tar.gz | tar xvf -

    Run the GUI:

        % ./ssvnc/Unix/ssvnc               (for Unix)

        % ./ssvnc/MacOSX/ssvnc             (for Mac OS X)

    The smaller file "ssvnc_no_windows-1.0.29.tar.gz"
    could have been used as well.

    On MacOSX you could also click on the SSVNC app icon in the Finder.

    On MacOSX if you don't like the Chicken of the VNC (e.g. no local
    cursors, no screen size rescaling, and no password prompting), and you
    have the XDarwin X server installed, you can set DISPLAY before starting
    ssvnc (or type DISPLAY=... in Host:Disp and hit Return).  Then our
    enhanced TightVNC viewer will be used instead of COTVNC.
    Update: there is now a 'Use X11 vncviewer on MacOSX' under Options ...


    If you want a SSH-only tool (without the distractions of SSL) run
    the command:

                sshvnc

    instead of "ssvnc".  Or click "SSH-Only Mode" under Options.
    Control-h will toggle between the two modes.


    If you want a simple VNC Terminal Services only mode (requires x11vnc
    on the remote server) run the command:

                tsvnc

    instead of "ssvnc".  Or click "Terminal Services" under Options.
    Control-t will toggle between the two modes.

    "tsvnc profile-name" and "tsvnc user@hostname" work too.


Unix/MacOSX Install:

    There is no standard install for the bundles, but you can make
    symlinks like so:

        cd /a/directory/in/PATH
        ln -s /path/to/ssvnc/bin/{s,t}* .

    Or put /path/to/ssvnc/bin, /path/to/ssvnc/Unix, or /path/to/ssvnc/MacOSX
    in your PATH.

    For the conventional source tarball it will compile and install, e.g.:

       gzip -dc ssvnc-1.0.29.src.tar.gz | tar xvf -
       cd ssvnc-1.0.29
       make config
       make all
       make PREFIX=/my/install/dir install

    then have /my/install/dir/bin in your PATH.



Windows:

    Unzip, using WinZip or a similar utility, the zip file:

        ssvnc-1.0.29.zip

    Run the GUI, e.g.:

        Start -> Run -> Browse

    and then navigate to

        .../ssvnc/Windows/ssvnc.exe

    select Open, and then OK to launch it.

    The smaller file "ssvnc_windows_only-1.0.29.zip"
    could have been used as well.

    You can make a Windows shortcut to this program if you want to.

    See the Windows/README.txt for more info.


    If you want a SSH-only tool (without the distractions of SSL) run
    the command:

                sshvnc.bat

    Or click "SSH-Only Mode" under Options.


    If you want a simple VNC Terminal Services only mode (requires x11vnc
    on the remote server) run the command:

                tsvnc.bat

    Or click "Terminal Services" under Options.  Control-t will toggle
    between the two modes.  "tsvnc profile-name" and "tsvnc user@hostname"
    work too.

     _________________________________________________________________

   You can read all of the SSVNC GUI's Online Help Text here.
     _________________________________________________________________

   The bundle unpacks a directory/folder named: ssvnc. It contains these
   programs to launch the GUI:
        Windows/ssvnc.exe        for Windows
        MacOSX/ssvnc             for Mac OS X
        Unix/ssvnc               for Unix

   (the Mac OS X and Unix launchers are simply links to the bin
   directory). See the README for more information.

   The SSH-Only mode launcher program has name sshvnc. The Terminal
   Services mode launcher program (assumes x11vnc 0.8.4 or later and Xvfb
   installed on the server machine) has name tsvnc.

   The Viewer SSL support is done via a wrapper script (bin/ssvnc_cmd
   that calls bin/util/ss_vncviewer) that starts up the STUNNEL tunnel
   first and then starts the TightVNC viewer pointed at that tunnel. The
   bin/ssvnc program is a GUI front-end to that script. See this FAQ for
   more details on SSL tunnelling. In SSH connection mode, the wrappers
   start up SSH appropriately.


   Memory Stick Usage: If you create a directory named "Home" in that
   toplevel ssvnc directory then that will be used as the base for
   storing VNC profiles and certificates. Also, for convenience, if you
   first run the command with "." as an argument (e.g. "ssvnc .") it will
   automatically create the "Home" directory for you. This is handy if
   you want to place SSVNC on a USB flash drive that you carry around for
   mobile use and you want the profiles you create to stay with the drive
   (otherwise you'd have to browse to the drive directory each time you
   load or save).

   One user on Windows created a BAT file to launch SSVNC and needed to
   do this to get the Home directory correct:
cd \ssvnc\Windows
start \ssvnc\Windows\ssvnc.exe

   (an optional profile name can be supplied to the ssvnc.exe line)

   WARNING: if you use ssvnc from an "Internet Cafe", i.e. some untrusted
   computer, please be aware that someone may have set up that machine to
   be capturing your keystrokes, etc.


   SSH-Only version: The command "sshvnc" can be run instead of "ssvnc"
   to get an SSH-only version of the tool:

   [sshvnc.gif]

   These also work: "sshvnc myprofile" and "sshvnc user@hostname". To
   switch from the regular SSVNC mode, click "SSH-Only Mode" under
   Options. This mode is less distracting if you never plan to use SSL,
   manage certificates, etc.


   Terminal Services Only: The command "tsvnc" can be run instead of
   "ssvnc" to get a "Terminal Services" only version of the tool:

   [tsvnc.gif]

   These also work: "tsvnc myprofile" and "tsvnc user@hostname". To
   switch from the regular SSVNC mode, click "Terminal Services" under
   Options.

   This mode requires x11vnc (0.9.3 or later) installed on the remote
   machine to find, create, and manage the user sessions. SSH is used to
   create the encrypted and authenticated tunnel. The Xvfb (virtual
   framebuffer X server) program must also be installed on the remote
   system. However tsvnc will also connect to a real X session (i.e. on
   the physical hardware) if you are already logged into the X session;
   this is a useful access mode and does not require Xvfb on the remote
   system.

   This mode should be very easy for beginner users to understand and
   use. On the remote end you only need to have x11vnc and Xvfb available
   in $PATH, and on the local end you just run something like:
   tsvnc myname@myhost.com

   (or start up the tsvnc GUI first and then enter myname@myhost.com and
   press "Connect").

   Normally the Terminal Services sessions created are virtual (RAM-only)
   ones (e.g. Xvfb, Xdummy, or Xvnc), however a nice feature is if you
   have a regular X session (i.e displaying on the physical hardware) on
   the remote machine that you are ALREADY logged into, then the x11vnc
   run from tsvnc will find it for you as well.

   Also, there is setting "X Login" under Advanced Options that allows
   you to attach to a real X server with no one logged in yet (i.e.
   XDM/GDM/KDM Login Greeter screen) as long as you have sudo(1)
   permission on the remote machine.

   Nice features to soon to be added to the tsvnc mode are easy CUPS
   printing (working fairly well) and Sound redirection (needs much work)
   of the Terminal Services Desktop session. It is easier in tsvnc mode
   because the entire desktop session can be started with the correct
   environment. ssvnc tries to handle the general case of an already
   started desktop and that is more difficult.


   Proxies: Web proxies, SOCKS proxies, and the UltraVNC repeater proxy
   are supported to allow the SSVNC connection to go through the proxy to
   the otherwise unreachable VNC Server. SSH gateway machines can be used
   in the same way. Read more about SSVNC proxy support here.


   Dynamic VNC Server Port determination: If you are running SSVNC on
   Unix and are using SSH to start the remote VNC server and the VNC
   server prints out the line "PORT=NNNN" to indicate which dynamic port
   it is using (x11vnc does this), then if you prefix the SSH command
   with "PORT=" SSVNC will watch for the PORT=NNNN line and uses ssh's
   built in SOCKS proxy (ssh -D ...) to connect to the dynamic VNC server
   port through the SSH tunnel. For example:
        VNC Host:Display     user@somehost.com
        Remote SSH Command:  PORT= x11vnc -find

   or "PORT= x11vnc -display :0 -localhost", etc. Or use "P= x11vnc ..."

   There is also code to detect the display of the regular Unix
   vncserver(1). It extracts the display (and hence port) from the lines
   "New 'X' desktop is hostname:4" and also "VNC server is already
   running as :4". So you can use something like:
        PORT= vncserver; sleep 15
or:     PORT= vncserver :4; sleep 15

   the latter is preferred because when you reconnect with it will find
   the already running one. The former one will keep creating new X
   sessions if called repeatedly.

   If you use PORT= on Windows, a large random port is selected instead
   and the -rfbport option is passed to x11vnc (it does not work with
   vncserver).



   Patches for Unix Tightvnc viewer:

   The rfbNewFBSize support allows the enhanced TightVNC Unix viewer to
   resize when the server does (e.g. "x11vnc -R scale=3/4" remote control
   command).

   The cursor alphablending is described here.

   The RealVNC ZRLE encoding is supported, in addition to some low colors
   modes (16bpp and 8bpp at 256, 64, and even 8 colors, for use on very
   slow connections). Greyscales are also enabled for the low color
   modes.

   The Popup menu (F8) is enhanced with the ability to change many things
   on the fly. F9 is added as a shortcut to toggle FullScreen mode.

   Client Side Caching: The x11vnc client-side caching is handled nicely
   by this viewer. The very large pixel cache below the actual display in
   this caching method is distracting. Our Unix VNC viewer will
   automatically try to autodetect the actual display height if the
   framebuffer is very tall (more than twice as high as it is wide). One
   can also set the height to the known value via -ycrop n, or use -ycrop
   -1 to force autodection. In fullscreen mode one is not possible to
   scroll down to the pixel cache region. In non-fullscreen mode the
   window manager frame is "shrink-wrapped" around the actual screen
   display. You can still scroll down to the pixel cache region. The
   scrollbars are set to be very thin (2 pixels) to be less distracting.
   Use the -sbwidth n to make them wider.

   Probably nobody is interested in the grabserver patch for old window
   managers when the viewer is in fullscreen mode... This and some other
   unfixed bugs have been fixed in our patches (fullscreen toggle works
   with KDE, -x11cursor has been fixed, and the dot cursor has been made
   smaller).

   From the -help output:
SSVNC Viewer (based on TightVNC viewer version 1.3.9)

Usage: vncviewer [<OPTIONS>] [<HOST>][:<DISPLAY#>]
       vncviewer [<OPTIONS>] [<HOST>][::<PORT#>]
       vncviewer [<OPTIONS>] exec=[CMD ARGS...]
       vncviewer [<OPTIONS>] fd=n
       vncviewer [<OPTIONS>] /path/to/unix/socket
       vncviewer [<OPTIONS>] -listen [<DISPLAY#>]
       vncviewer -help

<OPTIONS> are standard Xt options, or:
        -via <GATEWAY>
        -shared (set by default)
        -noshared
        -viewonly
        -fullscreen
        -noraiseonbeep
        -passwd <PASSWD-FILENAME> (standard VNC authentication)
        -user <USERNAME> (Unix login authentication)
        -encodings <ENCODING-LIST> (e.g. "tight,copyrect")
        -bgr233
        -owncmap
        -truecolour
        -depth <DEPTH>
        -compresslevel <COMPRESS-VALUE> (0..9: 0-fast, 9-best)
        -quality <JPEG-QUALITY-VALUE> (0..9: 0-low, 9-high)
        -nojpeg
        -nocursorshape
        -x11cursor
        -autopass

Option names may be abbreviated, e.g. -bgr instead of -bgr233.
See the manual page for more information.


Enhanced TightVNC viewer (SSVNC) options:

   URL http://www.karlrunge.com/x11vnc/ssvnc.html

   Note: ZRLE and ZYWRLE encodings are now supported.

   Note: F9 is shortcut to Toggle FullScreen mode.

   Note: In -listen mode set the env var. SSVNC_MULTIPLE_LISTEN=1
         to allow more than one incoming VNC server at a time.
         This is the same as -multilisten described below.  Set
         SSVNC_MULTIPLE_LISTEN=MAX:n to allow no more than "n"
         simultaneous reverse connections.

   Note: If the host:port is specified as "exec=command args..."
         then instead of making a TCP/IP socket connection to the
         remote VNC server, "command args..." is executed and the
         viewer is attached to its stdio.  This enables tunnelling
         established via an external command, e.g. an stunnel(8)
         that does not involve a listening socket.  This mode does
         not work for -listen reverse connections.

         If the host:port is specified as "fd=n" then it is assumed
         n is an already opened file descriptor to the socket. (i.e
         the parent did fork+exec)

         If the host:port contains a '/' it is interpreted as a
         unix-domain socket (AF_LOCAL insead of AF_INET)

        -multilisten  As in -listen (reverse connection listening) except
                    allow more than one incoming VNC server to be connected
                    at a time.  The default for -listen of only one at a
                    time tries to play it safe by not allowing anyone on
                    the network to put (many) desktops on your screen over
                    a long window of time. Use -multilisten for no limit.

        -acceptpopup  In -listen (reverse connection listening) mode when
                    a reverse VNC connection comes in show a popup asking
                    whether to Accept or Reject the connection.  The IP
                    address of the connecting host is shown.  Same as
                    setting the env. var. SSVNC_ACCEPT_POPUP=1.

        -acceptpopupsc  As in -acceptpopup except assume UltraVNC Single
                    Click (SC) server.  Retrieve User and ComputerName
                    info from UltraVNC Server and display in the Popup.

        -use64      In -bgr233 mode, use 64 colors instead of 256.
        -bgr222     Same as -use64.

        -use8       In -bgr233 mode, use 8 colors instead of 256.
        -bgr111     Same as -use8.

        -16bpp      If the vnc viewer X display is depth 24 at 32bpp
                    request a 16bpp format from the VNC server to cut
                    network traffic by up to 2X, then tranlate the
                    pixels to 32bpp locally.
        -bgr565     Same as -16bpp.

        -grey       Use a grey scale for the 16- and 8-bpp modes.

        -alpha      Use alphablending transparency for local cursors
                    requires: x11vnc server, both client and server
                    must be 32bpp and same endianness.

        -scale str  Scale the desktop locally.  The string "str" can
                    a floating point ratio, e.g. "0.9", or a fraction,
                    e.g. "3/4", or WxH, e.g. 1280x1024.  Use "fit"
                    to fit in the current screen size.  Use "auto" to
                    fit in the window size.  "str" can also be set by
                    the env. var. SSVNC_SCALE.

                    If you observe mouse trail painting errors, enable
                    X11 Cursor mode (either via Popup or -x11cursor.)

                    Note that scaling is done in software and so can be
                    slow and requires more memory.  Some speedup Tips:

                        ZRLE is faster than Tight in this mode.  When
                        scaling is first detected, the encoding will
                        be automatically switched to ZRLE.  Use the
                        Popup menu if you want to go back to Tight.
                        Set SSVNC_PRESERVE_ENCODING=1 to disable this.

                        Use a solid background on the remote side.
                        (e.g. manually or via x11vnc -solid ...)

                        If the remote server is x11vnc, try client
                        side caching: x11vnc -ncache 10 ...

        -ycrop n    Only show the top n rows of the framebuffer.  For
                    use with x11vnc -ncache client caching option
                    to help "hide" the pixel cache region.
                    Use a negative value (e.g. -1) for autodetection.
                    Autodetection will always take place if the remote
                    fb height is more than 2 times the width.

        -sbwidth n  Scrollbar width for x11vnc -ncache mode (-ycrop),
                    default is very narrow: 2 pixels, it is narrow to
                    avoid distraction in -ycrop mode.

        -nobell     Disable bell.

        -rawlocal   Prefer raw encoding for localhost, default is
                    no, i.e. assumes you have a SSH tunnel instead.

        -notty      Try to avoid using the terminal for interactive
                    responses: use windows for messages and prompting
                    instead.  Messages will also be printed to terminal.

        -sendclipboard  Send the X CLIPBOARD selection (i.e. Ctrl+C,
                        Ctrl+V) instead of the X PRIMARY selection (mouse
                        select and middle button paste.)

        -sendalways     Whenever the mouse enters the VNC viewer main
                        window, send the selection to the VNC server even if
                        it has not changed.  This is like the Xt resource
                        translation SelectionToVNC(always)

        -recvtext str   When cut text is received from the VNC server,
                        ssvncviewer will set both the X PRIMARY and the
                        X CLIPBOARD local selections.  To control which
                        is set, specify 'str' as 'primary', 'clipboard',
                        or 'both' (the default.)

        -graball    Grab the entire X server when in fullscreen mode,
                    needed by some old window managers like fvwm2.

        -popupfix   Warp the popup back to the pointer position,
                    needed by some old window managers like fvwm2.
        -sendclipboard  Send the X CLIPBOARD selection (i.e. Ctrl+C,
                        Ctrl+V) instead of the X PRIMARY selection (mouse
                        select and middle button paste.)

        -sendalways     Whenever the mouse enters the VNC viewer main
                        window, send the selection to the VNC server even if
                        it has not changed.  This is like the Xt resource
                        translation SelectionToVNC(always)

        -recvtext str   When cut text is received from the VNC server,
                        ssvncviewer will set both the X PRIMARY and the
                        X CLIPBOARD local selections.  To control which
                        is set, specify 'str' as 'primary', 'clipboard',
                        or 'both' (the default.)

        -graball    Grab the entire X server when in fullscreen mode,
                    needed by some old window managers like fvwm2.

        -popupfix   Warp the popup back to the pointer position,
                    needed by some old window managers like fvwm2.

        -grabkbd    Grab the X keyboard when in fullscreen mode,
                    needed by some window managers. Same as -grabkeyboard.
                    -grabkbd is the default, use -nograbkbd to disable.

        -bs, -nobs  Whether or not to use X server Backingstore for the
                    main viewer window.  The default is to not, mainly
                    because most Linux, etc, systems X servers disable
                    *all* Backingstore by default.  To re-enable it put

                        Option "Backingstore"

                    in the Device section of /etc/X11/xorg.conf.
                    In -bs mode with no X server backingstore, whenever an
                    area of the screen is re-exposed it must go out to the
                    VNC server to retrieve the pixels. This is too slow.

                    In -nobs mode, memory is allocated by the viewer to
                    provide its own backing of the main viewer window. This
                    actually makes some activities faster (changes in large
                    regions) but can appear to "flash" too much.

        -noshm      Disable use of MIT shared memory extension (not recommended
)

        -termchat   Do the UltraVNC chat in the terminal vncviewer is in
                    instead of in an independent window.

        -unixpw str Useful for logging into x11vnc in -unixpw mode. "str" is a
                    string that allows many ways to enter the Unix Username
                    and Unix Password.  These characters: username, newline,
                    password, newline are sent to the VNC server after any VNC
                    authentication has taken place.  Under x11vnc they are
                    used for the -unixpw login.  Other VNC servers could do
                    something similar.

                    You can also indicate "str" via the environment
                    variable SSVNC_UNIXPW.

                    Note that the Escape key is actually sent first to tell
                    x11vnc to not echo the Unix Username back to the VNC
                    viewer. Set SSVNC_UNIXPW_NOESC=1 to override this.

                    If str is ".", then you are prompted at the command line
                    for the username and password in the normal way.  If str is
                    "-" the stdin is read via getpass(3) for username@password.
                    Otherwise if str is a file, it is opened and the first line
                    read is taken as the Unix username and the 2nd as the
                    password. If str prefixed by "rm:" the file is removed
                    after reading. Otherwise, if str has a "@" character,
                    it is taken as username@password. Otherwise, the program
                    exits with an error. Got all that?

     -repeater str  This is for use with UltraVNC repeater proxy described
                    here: http://www.uvnc.com/addons/repeater.html.  The "str"
                    is the ID string to be sent to the repeater.  E.g. ID:1234
                    It can also be the hostname and port or display of the VNC
                    server, e.g. 12.34.56.78:0 or snoopy.com:1.  Note that when
                    using -repeater, the host:dpy on the cmdline is the repeate
r
                    server, NOT the VNC server.  The repeater will connect you.

                    Example: vncviewer ... -repeater ID:3333 repeat.host:5900
                    Example: vncviewer ... -repeater vhost:0 repeat.host:5900

                    Use, e.g., '-repeater SCIII=ID:3210' if the repeater is a
                    Single Click III (SSL) repeater (repeater_SSL.exe) and you
                    are passing the SSL part of the connection through stunnel,
                    socat, etc. This way the magic UltraVNC string 'testB'
                    needed to work with the repeater is sent to it.

     -rfbversion str Set the advertised RFB version.  E.g.: -rfbversion 3.6
                    For some servers, e.g. UltraVNC this needs to be done.

     -ultradsm      UltraVNC has symmetric private key encryption DSM plugins:
                    http://www.uvnc.com/features/encryption.html. It is assumed
                    you are using a unix program (e.g. our ultravnc_dsm_helper)
                    to encrypt and decrypt the UltraVNC DSM stream. IN ADDITION
                    TO THAT supply -ultradsm to tell THIS viewer to modify the
                    RFB data sent so as to work with the UltraVNC Server. For
                    some reason, each RFB msg type must be sent twice under DSM
.

     -mslogon user  Use Windows MS Logon to an UltraVNC server.  Supply the
                    username or "1" to be prompted.  The default is to
                    autodetect the UltraVNC MS Logon server and prompt for
                    the username and password.

                    IMPORTANT NOTE: The UltraVNC MS-Logon Diffie-Hellman
                    exchange is very weak and can be brute forced to recover
                    your username and password in a few seconds of CPU time.
                    To be safe, be sure to use an additional encrypted tunnel
                    (e.g. SSL or SSH) for the entire VNC session.

     -chatonly      Try to be a client that only does UltraVNC text chat. This
                    mode is used by x11vnc to present a chat window on the
                    physical X11 console (i.e. chat with the person at the
                    display).

     -env VAR=VALUE To save writing a shell script to set environment variables
,
                    specify as many as you need on the command line.  For
                    example, -env SSVNC_MULTIPLE_LISTEN=MAX:5 -env EDITOR=vi

     -noipv6        Disable all IPv6 sockets.  Same as VNCVIEWER_NO_IPV6=1.

     -noipv4        Disable all IPv4 sockets.  Same as VNCVIEWER_NO_IPV4=1.

     -printres      Print out the Ssvnc X resources (appdefaults) and then exit
                    You can save them to a file and customize them (e.g. the
                    keybindings and Popup menu)  Then point to the file via
                    XENVIRONMENT or XAPPLRESDIR.

     -pipeline      Like TurboVNC, request the next framebuffer update as soon
                    as possible instead of waiting until the end of the current
                    framebuffer update coming in.  Helps 'pipeline' the updates
.
                    This is currently the default, use -nopipeline to disable.

     -appshare      Enable features for use with x11vnc's -appshare mode where
                    instead of sharing the full desktop only the application's
                    windows are shared.  Viewer multilisten mode is used to
                    create the multiple windows: -multilisten is implied.
                    See 'x11vnc -appshare -help' more information on the mode.

                    Features enabled in the viewer under -appshare are:
                    Minimum extra text in the title, auto -ycrop is disabled,
                    x11vnc -remote_prefix X11VNC_APPSHARE_CMD: message channel,
                    x11vnc initial window position hints.  See also Escape Keys
                    below for additional key and mouse bindings.

     -escape str    This sets the 'Escape Keys' modifier sequence and enables
                    escape keys mode.  When the modifier keys escape sequence
                    is held down, the next keystroke is interpreted locally
                    to perform a special action instead of being sent to the
                    remote VNC server.

                    Use '-escape default' for the default modifier sequence.
                    (Unix: Alt_L,Super_L and MacOSX: Control_L,Meta_L)

    Here are the 'Escape Keys: Help+Set' instructions from the Popup Menu:

    Escape Keys:  Enter a comma separated list of modifier keys to be the
    'escape sequence'.  When these keys are held down, the next keystroke is
    interpreted locally to invoke a special action instead of being sent to
    the remote VNC server.  In other words, a set of 'Hot Keys'.

    To enable or disable this, click on 'Escape Keys: Toggle' in the Popup.

    Here is the list of hot-key mappings to special actions:

       r: refresh desktop  b: toggle bell   c: toggle full-color
       f: file transfer    x: x11cursor     z: toggle Tight/ZRLE
       l: full screen      g: graball       e: escape keys dialog
       s: scale dialog     +: scale up (=)  -: scale down (_)
       t: text chat                         a: alphablend cursor
       V: toggle viewonly  Q: quit viewer   1 2 3 4 5 6: UltraVNC scale 1/n

       Arrow keys:         pan the viewport about 10% for each keypress.
       PageUp / PageDown:  pan the viewport by a screenful vertically.
       Home   / End:       pan the viewport by a screenful horizontally.
       KeyPad Arrow keys:  pan the viewport by 1 pixel for each keypress.
       Dragging the Mouse with Button1 pressed also pans the viewport.
       Clicking Mouse Button3 brings up the Popup Menu.

    The above mappings are *always* active in ViewOnly mode, unless you set the
    Escape Keys value to 'never'.

    If the Escape Keys value below is set to 'default' then a default list of
    of modifier keys is used.  For Unix it is: Alt_L,Super_L and for MacOSX it
    is Control_L,Meta_L.  Note: the Super_L key usually has a Windows(TM) Flag
    on it.  Also note the _L and _R mean the key is on the LEFT or RIGHT side
    of the keyboard.

    On Unix   the default is Alt and Windows keys on Left side of keyboard.
    On MacOSX the default is Control and Command keys on Left side of keyboard.

    Example: Press and hold the Alt and Windows keys on the LEFT side of the
    keyboard and then press 'c' to toggle the full-color state.  Or press 't'
    to toggle the ultravnc Text Chat window, etc.

    To use something besides the default, supply a comma separated list (or a
    single one) from: Shift_L Shift_R Control_L Control_R Alt_L Alt_R Meta_L
    Meta_R Super_L Super_R Hyper_L Hyper_R or Mode_switch.


   New Popup actions:

        ViewOnly:                ~ -viewonly
        Disable Bell:            ~ -nobell
        Cursor Shape:            ~ -nocursorshape
        X11 Cursor:              ~ -x11cursor
        Cursor Alphablend:       ~ -alpha
        Toggle Tight/Hextile:    ~ -encodings hextile...
        Toggle Tight/ZRLE:       ~ -encodings zrle...
        Toggle ZRLE/ZYWRLE:      ~ -encodings zywrle...
        Quality Level            ~ -quality (both Tight and ZYWRLE)
        Compress Level           ~ -compresslevel
        Disable JPEG:            ~ -nojpeg  (Tight)
        Pipeline Updates         ~ -pipeline

        Full Color                 as many colors as local screen allows.
        Grey scale (16 & 8-bpp)  ~ -grey, for low colors 16/8bpp modes only.
        16 bit color (BGR565)    ~ -16bpp / -bgr565
        8  bit color (BGR233)    ~ -bgr233
        256 colors               ~ -bgr233 default # of colors.
         64 colors               ~ -bgr222 / -use64
          8 colors               ~ -bgr111 / -use8
        Scale Viewer             ~ -scale
        Escape Keys: Toggle      ~ -escape
        Escape Keys: Help+Set    ~ -escape
        Set Y Crop (y-max)       ~ -ycrop
        Set Scrollbar Width      ~ -sbwidth
        XGrabServer              ~ -graball

        UltraVNC Extensions:

          Set 1/n Server Scale     Ultravnc ext. Scale desktop by 1/n.
          Text Chat                Ultravnc ext. Do Text Chat.
          File Transfer            Ultravnc ext. File xfer via Java helper.
          Single Window            Ultravnc ext. Grab and view a single window.
                                   (select then click on the window you want).
          Disable Remote Input     Ultravnc ext. Try to prevent input and
                                   viewing of monitor at physical display.

        Note: the Ultravnc extensions only apply to servers that support
              them.  x11vnc/libvncserver supports some of them.

        Send Clipboard not Primary  ~ -sendclipboard
        Send Selection Every time   ~ -sendalways

   Nearly all of these can be changed dynamically in the Popup menu
   (press F8 for it):

   [viewer_menu.gif] [unixviewer.jpg]

     _________________________________________________________________

   Windows:

   For Windows, SSL Viewer support is provided by a GUI Windows/ssvnc.exe
   that prompts for the VNC display and then starts up STUNNEL followed
   by the Stock TightVNC Windows Viewer. Both are bundled in the package
   for your convenience. The GUI has other useful features. When the
   connection is finished, you will be asked if you want to terminate the
   STUNNEL program. For SSH connections from Windows the GUI will use
   PLINK instead of STUNNEL.

   Unix and Mac OS X:

   Run the GUI (ssvnc, see above) and let me know how it goes.
     _________________________________________________________________

   Hopefully this tool will make it convenient for people to help test
   and use the built-in SSL support in x11vnc. Extra testing of this
   feature is much appreciated!! Thanks.

   Please Help Test the newly added features:
     * Automatic Service tunnelling via SSH for CUPS and SMB Printing
     * ESD/ARTSD Audio
     * SMB (Windows/Samba) filesystem mounting

   These allow you to print from the remote (VNC Server) machine to local
   printers, listen to sounds (with some limitations) from the remote VNC
   Server machine, and to mount your local Windows or Samba shares on the
   remote VNC Server machine. Basically these new features try to
   automate the tricks described here:
    http://www.karlrunge.com/x11vnc/faq.html#faq-smb-shares
    http://www.karlrunge.com/x11vnc/faq.html#faq-cups
    http://www.karlrunge.com/x11vnc/faq.html#faq-sound
     _________________________________________________________________

   Downloading: Downloads for this project are hosted at Sourceforge.net.

   Choose the archive file bundle that best suits you (e.g. no source
   code, windows only, unix only, zip, tar etc).

   A quick guide:

      On some flavor of Unix, e.g. Linux or Solaris? Use
   "ssvnc_unix_only" (or "ssvnc_no_windows" to recompile).
      On Mac OS X? Use "ssvnc_no_windows".
      On Windows? Use "ssvnc_windows_only".
  ssvnc_windows_only-1.0.28.zip      Windows Binaries Only.  No source included
 (6.2MB)
  ssvnc_no_windows-1.0.28.tar.gz     Unix and Mac OS X Only. No Windows binarie
s.  Source included. (10.1MB)
  ssvnc_unix_only-1.0.28.tar.gz      Unix Binaries Only.     No source included
. (7.2MB)
  ssvnc_unix_minimal-1.0.28.tar.gz   Unix Minimal.  You must supply your own vn
cviewer and stunnel. (0.2MB)

  ssvnc-1.0.28.tar.gz                All Unix, Mac OS X, and Windows binaries a
nd source TGZ. (16.1MB)
  ssvnc-1.0.28.zip                   All Unix, Mac OS X, and Windows binaries a
nd source ZIP. (16.4MB)
  ssvnc_all-1.0.28.zip               All Unix, Mac OS X, and Windows binaries a
nd source AND full archives in the zip dir. (19.2MB)


   Here is a conventional source tarball:
  ssvnc-1.0.28.src.tar.gz            Conventional Source for SSVNC GUI and Unix
 VNCviewer  (0.5MB)

   it will be of use to those who do not want the SSVNC
   "one-size-fits-all" bundles. For example, package/distro maintainers
   will find this more familiar and useful to them (i.e. they run: "make
   config; make all; make install"). Note that it does not include the
   stunnel source, and so has a dependency that the system stunnel is
   installed.

   Read the README.src file for more information on using the
   conventional source tarball.


   Note: even with the Unix bundles, e.g. "ssvnc_no_windows" or
   "ssvnc_all", you may need to run the "./build.unix" script in the top
   directory to recompile for your operating system.

   Here are the corresponding 1.0.29 development bundles (Please help
   test them):

  ssvnc_windows_only-1.0.29.zip
  ssvnc_no_windows-1.0.29.tar.gz
  ssvnc_unix_only-1.0.29.tar.gz
  ssvnc_unix_minimal-1.0.29.tar.gz

  ssvnc-1.0.29.tar.gz
  ssvnc-1.0.29.zip
  ssvnc_all-1.0.29.zip

  ssvnc-1.0.29.src.tar.gz            Conventional Source for SSVNC GUI and Unix
 VNCviewer  (0.5MB)


   For any Unix system, a self-extracting and running file for the
   "ssvnc_unix_minimal" package is here: ssvnc. Save it as filename
   "ssvnc", type "chmod 755 ./ssvnc", and then launch the GUI via typing
   "./ssvnc". Note that this "ssvnc_unix_minimal" mode requires you
   install the "stunnel" and "vncviewer" programs externally (for
   example, install your distros' versions, e.g. on debian: "apt-get
   install stunnel4 xtightvncviewer".) It will work, but many of the
   SSVNC features will be missing.

   Previous releases:
      Release 1.0.18 at Sourceforge.net
      Release 1.0.19 at Sourceforge.net
      Release 1.0.20 at Sourceforge.net
      Release 1.0.21 at Sourceforge.net
      Release 1.0.22 at Sourceforge.net
      Release 1.0.23 at Sourceforge.net
      Release 1.0.24 at Sourceforge.net
      Release 1.0.25 at Sourceforge.net
      Release 1.0.26 at Sourceforge.net
      Release 1.0.27 at Sourceforge.net
      Release 1.0.28 at Sourceforge.net


   Please help test the UltraVNC File Transfer support in the native Unix
   VNC viewer! Let us know how it went.

   Current Unix binaries in the archives:
    Linux.i686
    Linux.x86_64
    Linux.ppc64    X (removed)
    Linux.alpha    X (removed)
    SunOS.sun4u
    SunOS.sun4m
    SunOS.i86pc
    Darwin.Power.Macintosh
    Darwin.i386
    HP-UX.9000     X (removed)
    FreeBSD.i386   X (removed)
    NetBSD.i386    X (removed)
    OpenBSD.i386   X (removed)

   (some of these are out of date, marked with 'X' above, because I no
   longer have access to machines running those OS's. Use the
   "build.unix" script to recompile on your system).

   Note: some of the above binaries depend on libssl.so.0.9.7, whereas
   some recent distros only provide libssl.so.0.9.8 by default (for
   compatibility reasons they should install both by default but not all
   do). So you may need to instruct your distro to install the 0.9.7
   library (it is fine to have both runtimes installed simultaneously
   since the libraries have different names). Update: I now try to
   statically link libssl.a for all of the binaries in the archive.

   You can also run the included build.unix script to try to
   automatically build the binaries if your OS is not in the above list
   or the included binary does not run properly on your system. Let me
   know how that goes.
     _________________________________________________________________

   IMPORTANT: there may be restrictions for you to download, use, or
   redistribute the above because of cryptographic software they contain
   or for other reasons. Please check out your situation and information
   at the following and related sites:
        http://stunnel.mirt.net
        http://www.stunnel.org
        http://www.openssl.org
        http://www.chiark.greenend.org.uk/~sgtatham/putty/
        http://www.tightvnc.com
        http://www.realvnc.com
        http://sourceforge.net/projects/cotvnc/
     _________________________________________________________________

   README: Here is the toplevel README from the bundle.
	
=======================================================================