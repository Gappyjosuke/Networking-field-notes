[← Back to May Archive](/README.md)

# One Website Refusing To Open Somehow Turned Into Me Discovering My ISP Was Quietly Lying Through DNS

On that day I was trying to surf around internet with random websites, but some of the website simply refused to load.

The browser kept showing:

    "The connection has timed out"

At first I didn't really think much about it because websites randomly go down all the time. So I assumed maybe:
- the server of the website was overloaded
- the site was temporarily dead
- my internet connection itself was unstable

Pretty normal assumptions.

But then something strange happened.

The moment I turned ON my VPN…..

the website suddenly opened perfectly fine.

That instantly changed the entire situation because now the problem no longer looked like:

    website broken

Instead it became:

    why does the website work only when VPN is ON?

That was the first important clue because if the website works through VPN, then the website itself is probably completely alive somewhere on the internet.

Something else in the middle had to be behaving differently.

---

## Thinking About What Actually Changes When VPN Gets Enabled

Before this situation I mostly thought about VPNs as simple privacy tools or IP changers.

But this problem forces you to think more deeply about what a VPN is actually doing underneath.

Normally internet traffic roughly behaves like this:

<div align="center">

My PC  
↓  
ISP 

↓   
Internet  
↓  
Website  

</div>

But with VPN enabled:

<div align="center">

My PC  
↓  
Encrypted VPN Tunnel  
↓  
VPN Server  
↓  
Internet  
↓  
Website  

</div>

The VPN basically becomes a middle layer between me and the internet.

That means the ISP no longer directly handles certain parts of the communication the same way as before.

That immediately made me suspicious about the ISP layer itself.

Because:

    VPN ON  = works
    VPN OFF = fails

Something between my PC and the website was clearly changing.

---

## Breaking The Connection Into Layers

Instead of randomly trying fixes, I thought lets break the connection process into layers.

Something like this:

<div align="center">

Browser  
↓  
DNS Lookup  
↓  
Windows Networking  
↓  
Router / ISP  
↓  
Website Server  

</div>

The important realization here was that if even one layer fails, the website may never open at all.

So the entire investigation slowly became:

    which exact layer is failing?

That was when I started looking into DNS.

---

## Using nslookup To Inspect DNS Resolution

I ran:

    nslookup <website-that-i-am-trying-to-access>

At first I didn't fully understand what this command actually did underneath.

Later I realized it directly asks the configured DNS server:

    "What IP address belongs to this domain?"

The output immediately looked suspicious.

    C:\>nslookup <website-that-i-am-trying-to-access>
    Server:  bsnl-dns.local
    Address:  117.xxx.xxx.60

    Non-authoritative answer:
    Name:    <website-that-i-am-trying-to-access>
    Addresses:  117.xxx.xxx.65

The DNS server belonged to my ISP.

But the strange part was that the IP address returned for the website ALSO belonged to the ISP range itself.

That felt extremely wrong.

Why would some random website suddenly resolve back into infrastructure owned by the ISP?

That was the moment things slowly started clicking together in my head.

The situation started looking less like:

    website failure

and more like:

    the ISP DNS server is returning incorrect answers

---

## Understanding What DNS Actually Does

so lets start with start with understanding what DNS really is underneath the internet.

Humans remember websites using names like:

    google.com
    youtube.com
    example.com

But computers do not communicate using names.

They communicate using IP addresses.

So before opening any website, the operating system first asks:

    "Which IP address belongs to this domain name?"

That translation process is called DNS resolution.

Which means DNS is basically the internet's phonebook.

Normally the process should behave like this:

<div align="center">

You Ask DNS  
↓  
"Where is this website?"  
↓  
DNS Returns Real Server IP  

</div>

But now the behavior looked more like:

<div align="center">

You Ask ISP DNS  
↓  
"Where is this website?"  
↓  
ISP lies to you and Returns Different IP  
↓  
Browser Connects Wrong Destination  
↓  
Connection Times Out  

</div>

The website itself was never actually dead.

My system was simply being redirected toward the wrong destination entirely.

That realization honestly felt so annoying because until as an normal person without knowing these stuffs most of us mostly imagined ISPs as simple internet providers.

I never really thought about the fact that they also control the DNS systems many users automatically trust by default.

---

## Testing The Theory By Changing DNS Servers

At this point I wanted to verify whether DNS was truly the problem.

So I changed my DNS server from the ISP-provided DNS to Cloudflare DNS:

    1.1.1.1
    1.0.0.1

Then I cleared the old DNS cache using:

    ipconfig /flushdns

And suddenly…

the website opened instantly.

No VPN required anymore.

That basically confirmed the entire thing.

The problem was never:
- the browser
- the website itself
- the internet connection

The issue existed specifically at the DNS resolution layer.

---

## But why The VPN Was Actually Fixing The Problem

After understanding what DNS does, the VPN behavior finally makes complete sense.

When VPN was OFF:

<div align="center">

My PC  
↓  
ISP DNS  
↓  
Intentionally gives Incorrect DNS Response  
↓  
Wrong Destination  
↓  
Website Fails  

</div>

But when VPN was ON:

<div align="center">

My PC  
↓  
Encrypted VPN Tunnel  
↓  
VPN DNS / VPN Server  
↓  
Correct DNS Response  
↓  
Website Opens  

</div>

The VPN was not magically fixing the internet.

It was simply bypassing the ISP-controlled DNS system entirely.

That was probably the most interesting realization from this entire investigation.

---

## Realizing How Powerful DNS Actually Is

Another thing I realized is how powerful DNS really is underneath the internet.

Because whoever controls DNS can:
- redirect traffic
- block websites
- inject fake addresses
- censor domains
- monitor lookups

without ever touching the real website server itself.

Most users would simply assume:

    "The website is down."

when the real problem might actually be:

    "The DNS response itself was manipulated."

That realization completely changed how I think about internet connectivity problems.

---

[← Go to Previous Archive](/Journal/ARP/README.md)
---

[Go to Next Archive →](/Journal/Ethernet_l2/README.md)
---
