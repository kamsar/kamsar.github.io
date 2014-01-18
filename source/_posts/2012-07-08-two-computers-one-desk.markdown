---
layout: post
title: "Two computers, one desk"
date: 2012-07-08 12:15:11 -0800
comments: true
categories: 
---
I have a small desk, a PC, and a Mac mini. With only enough room for two monitors, I wanted to be able to use both computers effectively but traditional KVMs were too clunky and VNC too unreliable. I happened across an interesting piece of software called <a href="http://synergy-foss.org/" target="_blank">Synergy</a> that acts as a network-based virtual keyboard and mouse and set to making it integrate my PC and Mac use. The setup also works great to be able to test in Mac browsers and Windows browsers without any fuss.<br><br>
Synergy is quite an accomplishment. You select a machine to be the 'server' (i.e. the physical keyboard/mouse host), and install the Synergy client on the other machines. Then the clients connect to the server, and you can literally move the server's mouse straight onto the client machine's screen - just as if the client was a second monitor connected directly to the server. The really awesome part is that Synergy works on Windows, Mac, or Linux - so you could technically set up three monitors with a Mac, Linux, and Windows on each from three separate physical computers. Almost like virtualization on physical hardware :)<br><br>
Once I had Synergy working, I set about handling power management efficiently. The Mac mini has a power button in a very inconvenient location, so manually booting both computers and shutting down both computers was a pain in the rear. I found a method using a smart power strip and a shutdown script on the Windows machine to execute a graceful shutdown over SSH to the Mac whenever I told the Windows machine to shut down. Then, when I boot the PC, the smart power strip kicks in and also turns on the Mac. It works quite well.<br><br>
Sound fun? Read on. <i>Note: these instructions are written for relatively experienced users.</i><br><br><b>Prerequisites:</b><br><br><ul><li>A smart power strip (I used this <a href="http://www.amazon.com/Belkin-Conserve-Socket-Energy-Saving-Outlet/dp/B003P2UMQ2/" target="_blank">Belkin model</a>)</li>
<li>A Windows machine</li>
<li>A Mac (this works with Linux, other Windows, etc as well, but this guide focuses on Windows to Mac - technically you don't really need a Windows machine at all)</li>
<li>At least two monitors</li>
<li>Decent network bandwidth between the computers. I had some responsiveness issues with the Mac on 802.11g WiFi, but none at all once I put it on gigabit ethernet.</li>
</ul><div>
<b>Synergy Setup</b></div>
<div>
Following these instructions will get the keyboard and mouse of the Windows machine to be able to interface with the Mac's screen.</div>
<div>
<ul><li>Set up both computers as separate entities, connecting each to their own monitors. For initial setup, you'll want a keyboard/mouse on each (or you could swap them if you had to)</li>
<li>Install Synergy on the Windows machine, and select Server mode.</li>
<ul><li>It's simplest to install it as a service (which is the default) so it's always auto started</li>
<li>Add a Windows firewall exception for TCP port 24800 to allow remote connections to Synergy.</li>
<li>Open the Synergy application. Click the Configure Server button, then drag the screen icon to where the other computer's monitor is physically compared to the main computer.</li>
<li>Double click the screen and set the screen name to the other computer's name (probably something like "joes-mac-mini"). You can also choose an arbitrary name if desired, but the names must match on server and client settings.</li>
</ul><li>On the Mac, install Synergy in client mode. Give it the name (or IP address) of the other computer as the server name.</li>
<li>At this point, if all is well, you should be able to use the Windows machine's mouse and keyboard on the Mac's screen just as if it was a second monitor - just slide the mouse off the edge of the Windows monitor and it should turn into the Mac pointer on the Mac's monitor.</li>
<ul><li>If you have trouble, make sure the screen name is the same on both the Windows screen definitions and the Mac's screen name (in Synergy's Preferences screen)</li>
</ul><li>Configure Synergy to start automatically on the Mac <a href="http://synergy2.sourceforge.net/autostart.html" target="_blank">using the official instructions</a> (I used method #3, setting it as a Startup Item. This option only works well if you have automatic login enabled on the Mac.)</li>
</ul></div>
<div>
<b>Power Management Setup</b></div>
<div>
Following these instructions will get the Mac to shut down in tandem with the Windows machine as well as boot in tandem. The general approach is to configure a shutdown script on Windows that uses SSH to connect to the Mac and issue a shutdown command. <i>Important: these instructions suggest some things that are not exactly secure to synchronize shutdowns. I wouldn't suggest configuring this way unless you are aware of the implications of things like storing your password unencrypted in a file on disk. If you have suggestions on making these instructions more secure, I'd love to hear it.</i></div>
<div>
<ul><li>Shut down both computers</li>
<li>Plug the Windows machine into the smart power strip's Master socket. This will cause the other sockets on the power strip to be turned off when the Windows machine shuts down (or goes to sleep).</li>
<li>Plug the Mac and the monitors into the slave sockets on the smart power strip.</li>
<li>On the Mac, open the Sharing preferences pane and enable Remote Login (SSH)</li>
<li>On the Mac, open a Terminal window</li>
<ul><li>Create a script that will shut down the Mac. Place it in your home directory and call it shutdown.sh. The script will invoke a special shutdown mode that causes the Mac to boot when it gets power reapplied to it (shutdown -u), then do nothing for a while to wait for the shutdown to complete. It will look something like this:
<pre>echo <i>yourmacuserspassword</i> | sudo -S shutdown -h -u now
sleep 60</pre>
</li>
<li><i>Note: storing your password in a shell script is not very secure. The shutdown command must be run as root, hence the password requirement. There are AppleScript APIs that can run without root, but they don't seem to have the equivalent to the -u option.</i></li>
<li>Make the script executable by running chmod u+x shutdown.sh</li>
<li>Test the script by running ./shutdown.sh <i>This will shut down the Mac if it works right. Save your work first.</i></li>
</ul><li>On the Windows machine, download <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">Plink</a>, a command-line SSH client</li>
<li>On the Windows machine, create a folder to store Plink and your shutdown script. I used "C:\Mac" and will refer to that from here on.</li>
<ul><li>Open the security settings to the folder and grant the Everyone group Read and Execute access (<i>You could probably be more precise with this ACL but this will work</i>)</li>
<li>Copy the downloaded Plink to the folder</li>
<li>Create a new batch file and call it shutdown.cmd. This file will use Plink to connect to the Mac over SSH and execute the shell script we created previously. The batch file will look something like this: <pre>c:\mac\plink.exe <i>yourmacusername</i>@<i>your-macs-name</i> -pw <i>your-mac-users-password</i> /users/<i>yourmacusername</i>/shutdown.sh</pre>
</li>
<li><i>Note: you could hypothetically secure this SSH call from needing the username/password by <a href="http://xiix.wordpress.com/2007/03/31/how-to-set-up-public-key-authentication-pka-on-your-mac/" target="_blank">configuring public-key SSH authentication</a> on the Mac. I didn't try it, but it should work.</i></li>
<li>Test the script by running it. The Mac should shut itself down after a few seconds.</li>
</ul><li>On the Windows machine, open the Local Group Policy editor by typing Win+R and running gpedit.msc.</li>
<ul><li>Expand the Computer Configuration item, then the Windows Settings item</li>
<li>Click Scripts (Startup/Shutdown), then double click on Shutdown in the main pane</li>
<li>Click Add, then browse to your c:\mac\shutdown.cmd script</li>
<li>Save/Close all dialogs and the Group Policy editor</li>
</ul><li>Shut down the Windows machine. While the Windows machine is displaying "Shutting down...," the Mac should turn itself off followed very shortly by the Windows machine.</li>
<li>Wait for the smart power strip to turn off, then turn the Windows machine back on. When the smart power strip kicks in power to the Mac, it will also boot. </li>
<li>Congratulations, you have an ungodly hybrid of a machine guaranteed to disgust both Mac and Windows fanbois.</li>
<ul></ul></ul></div>
Happy hacking.