---
layout: post
title:  "Temporary fix for CERT VU#582384 / CVE-2016-6277 vulnerability for various Netgear routers (including R6400, R7000, R8000 and similar)"
tag: 	security
---

Like many people, I was affected by the recently disclosed code injection vulnerability (<a href="https://www.kb.cert.org/vuls/id/582384" target="_blank">CERT VU#582384</a>, [CVE-2016-6277/CVE-2016-582384](https://nvd.nist.gov/vuln/detail/CVE-2016-6277)) that affects Netgear R7000 ("Nighthawk") and R6400 routers. As Netgear haven't bothered to publish a quick patch for this gaping hole in their devices, I looked for a simple fix myself. This posts discusses the simple one-step process and the details. Depending on your tech skills, this will take you anywhere between five seconds and one minute.

<img src="http://www.sj-vs.net/wp-content/uploads/2016/12/netgear-R7000.jpg" alt="Netgear R7000" width="800" height="587" class="size-full wp-image-336" /> Netgear R7000


<p><strong>Update 13/12/2016</strong>: months after first having been notified of this problems, and four days after wide-spread media attention, Netgear has <em>finally</em> released beta firmwares to fix this vulnerability for the <a href="http://kb.netgear.com/000036454/R6400-Firmware-Version-1-0-1-18-Beta" target=_blank>R6400</a>, <a href="http://kb.netgear.com/000036453/R7000-Firmware-Version-1-0-7-6-Beta" target=_blank>R7000</a>, and <a href="http://kb.netgear.com/000036455/R8000-Firmware-Version-1-0-3-26-Beta" target=_blank>R8000</a>. More info here: <a href="http://kb.netgear.com/000036386/CVE-2016-582384" target=_blank>http://kb.netgear.com/000036386/CVE-2016-582384</a>.</p>

<center><h4><span style="color: #AA0000">Only continue reading if there is no <a href="http://kb.netgear.com/000036386/CVE-2016-582384" target=_blank>beta firmware</a> available for your device</span></h4></center>

This fix for the CERT VU#582384 vulnerability actually employs the vulnerability itself to forcefully stop the web server that is at the core of the vulnerability. This unfortunately means that (1) this fix will only work until you reboot (i.e., switch off/on) your router, and (2) you won't be able to access your router's settings panel via your web browser. On the plus side: none of the instructions below will make any <em>permanent</em> changes to your router configuration. Hopefully Netgear will soon come with a real fix for this issue.
<h2>tl;dr</h2>
<ol>
<li>(optional) verify that your router is affected by going to this URL: 
<code>http://[router-address]/cgi-bin/;uname$IFS-a</code>
If that shows you anything but an error (or an empty page): you're affected. If you're unsure: please read the detailed explanation below.</li>
<li>Point your browser to the following URL to terminate the web server process (which facilitates the vulnerability) on your router:
<code>http://[router-address]/cgi-bin/;killall$IFS'httpd'</code></li>
<li>(optional) verify that the URL in step (1) is no longer accessible</li>
</ol>

Did this fix save you from being hacked? Please consider <a href="#coffee">buying me a coffee</a>, and <a href="https://twitter.com/bas_van_schaik" target=_blank>do say hi on Twitter</a>.


<h2>Detailed explanation</h2>
Wherever I write <code>[router-address]</code> in an URL, you should replace that with the actual address of your router. For many people the magic word <code>routerlogin.net</code> will work. If that doesn't work: it's typically that's something like <code>192.168.0.1</code>. If you're on Mac or Linux, you can find this out by typing 'route -n' in a console (terminal) window. That will show you a table ‐ look for the value in the <code>Gateway</code> column that belongs to <code>Destination</code> <code>0.0.0.0</code> like in the picture below:

<img class="wp-image-303 size-full" src="http://www.sj-vs.net/wp-content/uploads/2016/12/ifconfig.png" alt="Run 'ifconfig' to find out the IP address of your Netgear router" width="897" height="190" /> Run 'ifconfig' to find out the IP address of your Netgear router

<h4 id="step1">Step 1 (optional): verify you're vulnerable</h4>
Open your browser and visit the following address:
<center><code>http://[router-address]/cgi-bin/;uname$IFS-a</code><br>
(For most people, this URL will work: <a href="http://www.routerlogin.net/cgi-bin/;uname$IFS-a" target=_blank>http://www.routerlogin.net/cgi-bin/;uname$IFS-a</a>)</center>&nbsp;

If a web page appears (which is not an error): you're vulnerable. In my case, the page contains a text that starts with: <code>Linux R7000 2.6.36.4brcmarm+ (...)</code>.

<h4>Step 2: terminate the web server process on your router</h4>
This is when you actually need to exploit the vulnerability in the router to simply terminate the web server process (which facilitates the remote vulnerability) on the router. Point your browser to the following URL:

<center>
<code>http://[router-address]/cgi-bin/;killall$IFS'httpd'</code><br>
(For most people, this URL will work: <a href="http://www.routerlogin.net/cgi-bin/;killall$IFS'httpd'" target=_blank>http://www.routerlogin.net/cgi-bin/;killall$IFS'httpd'</a>)
</center>&nbsp;

This will very likely give you a 'web server not available' error, but it will have stopped (killed) the web server process. This means that it is now impossible to exploit the vulnerability. Note that it is now also impossible to use the web configuration interface of your router (until you next turn it off and on again). You can double-check whether you're vulnerable by checking the URL in <a href="#step1">step 1</a>.

You are now safe from the CERT VU#582384 vulnerability. But don't forget: after turning your router off and on again (or a power cut!), the web server process will start again, and you will be vulnerable again!

<h2 id="coffee">Did this fix help you?</h2>
Did this fix save you from being hacked? Please consider buying me a coffee, and <a href="https://twitter.com/bas_van_schaik" target=_blank>do say hi on Twitter</a>.


<h2>List of affected devices</h2>
The list of affected devices seems to be growing. These are the devices I've heard about:
<ul>
<li><strong>R6250</strong> (AC1600): confirmed by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R6400</strong> (AC1750): confirmed by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R6700</strong> Nighthawk (AC1750): confirmed by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R7000</strong> Nighthawk (AC1900, AC2300): confirmed (by myself and Netgear <a href="#cite3-netgear">[3]</a>)</li>
<li><strong>R7100LG</strong> Nighthawk: confirmed by Netgear <a href="#cite3-netgear">[3]</a>)</li>
<li><strong>R7500</strong> Nighthawk X4 (AC2350): confirmed <a href="#cite2-kalypto">[2]</a> and Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R7800</strong> Nighthawk X4S(AC2600): confirmed <a href="#cite2-kalypto">[2]</a>, <em>not</em> by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R7900</strong> Nighthawk: confirmed by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R8000</strong> Nighthawk (AC3200): confirmed by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R8500</strong> Nighthawk X8 (AC5300): confirmed <a href="#cite2-kalypto">[2]</a>, <em>not</em> by Netgear <a href="#cite3-netgear">[3]</a></li>
<li><strong>R9000</strong> Nighthawk X10 (AD7200): confirmed <a href="#cite2-kalypto">[2]</a>, <em>not</em> by Netgear <a href="#cite3-netgear">[3]</a></li>
</ul>

It appears that most custom firmwares (e.g. <a href="https://advancedtomato.com/" target=_blank>AdvancedTomato</a>) are not vulnerable. If you want to be sure: just <a href="#step1">check step 1</a>! If you have more information, please let me know in the comments below. I've had at least one report of a user who experienced that the French version of his firmware was unaffected (R7000, v1.0.7.2_1.1.93), but the English version was in fact vulnerable.

<h2>References</h2>
[1] <a id="cite1-cert" href="https://www.kb.cert.org/vuls/id/582384">Cernegie Mellon University CERT Vulnerability Notes Database: VU #582384</a>
[2] <a id="cite2-kalypto" href="https://kalypto.org/research/netgear-vulnerability-expanded/">Kalypto (In)Security: NetGear Vulnerability Expanded</a>
[3] <a id="cite3-netgear" href="http://kb.netgear.com/000036386/CVE-2016-582384" target=_blank>Netgear: Security Advisory for VU 582384</a> (last version seen: 13/12/2016 12:00 noon GMT)


(I've closed the comments now Netgear have released a (beta) firmware to resolve this vulnerability. If you <em>really</em> need to contact me, <a href="https://twitter.com/bas_van_schaik" target=_blank>come and find me on Twitter</a>)
