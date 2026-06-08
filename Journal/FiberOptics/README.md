[← Back to Archive](/README.md)

# How Staring At Fiber Optic Connectors Accidentally Led Me To A Reddit Fight About 45-Second Server Hacks

I was writing down notes about the physical layer of the internet because I wanted to understand how data actually travels through glass strands. I was specifically looking at different types of fiber optic connectors like Subscriber Connectors (SC) and Lucent Connectors (LC).

I was trying to wrap my head around the pure logic of it. Like, how do you prevent light pulses from leaking out when a cable bends? 

Then I learned about a concept called:

    Total Internal Reflection

Basically, the cable has an inner glass core and an outer cladding layer. When light hits the boundary at a shallow angle, it bounces completely back inward instead of passing through. It’s basically a non-stop mirror maze for light.

After looking at how these cables connect to data centers using massive MPO plugs, I wanted to see how real-world Internet Service Providers actually deliver this connection straight to residential homes. 

So I started searching for stuff about:

    FTTN (Fiber to the Node)
    FTTB (Fiber to the Building)
    FTTH (Fiber to the Home)

I wanted to see real user experiences with raw FTTH setups, so I ended up searching on Reddit under some threads discussing local fiber providers. 

That is where I randomly came across this absolute trainwreck of an argument between two people.

The first guy dropped a comment that immediately sounded terrifying:

> **URPissingMeOff:** Don't ever hook a computer directly to the internet unless you have some decent networking skills and know how to set up a firewall correctly. 90% of internet traffic is hackbot networks, so it's a lot better to have a router dealing with all that rather than burning up CPU cycles on your computer. Plus you will eventually may want to provide internet access to other devices - second computer, game system, smart TV, internet-of-things device, cell phone, random guests/family wandering by, etc

Then this other guy immediately replied to point out how ridiculous that number sounded:

> **Gorilla_Flavored:** What do you mean, "90% of all internet traffic is hackbot networks"? You'll have to forgive me for thinking that this figure of 90% seems ridiculous and doesn't seem possible, I've never once, not in my life heard of the name of a single bot or bot network or the name of someone or a company who is running a botnet as described... Do you mean to tell me that these "hackbot networks" consume 9 times as much internet bandwith as every single person in the world? Who are the people running these hackbot networks? Considering that these theoretical, yet very tangible real life subscribers to real and actual internet service providers pay for these services to the tune of billions of dollars a year, the followup question I guess would be, who's paying for all of that?

But the first guy didn't back down at all:

> **URPissingMeOff:** Exactly that. Every single port number (65k of them) on every single IPV4 address ( 4 billion of them) on the internet is scanned multiple times a day by massive bot networks looking for vulnerabilities. In addition, every search engine and SEO "agency" is scanning every web port multiple times a day looking for new or updated content. This happening 24/7. If you put a new installation of an old vulnerable OS like the original version of Windows 2k on the internet with no firewall, it will be compromised in roughly 45 seconds. Take any computer, turn off the firewall, and plug it directly into a raw internet connection with no router in front of it. Run wireshark or a similar scanner for an hour.

Reading this completely sidetracked me. I stopped looking at fiber cables and started asking myself logical questions about this debate:

    How can bots use 9 times more bandwidth than real humans?
    Who is paying the billions of dollars for that data?
    Will a raw PC really get compromised in 45 seconds?
    Is the internet actually that lawless underneath?

So I stopped and started breaking down their arguments piece by piece to figure out what the realistic middle ground actually was.

First I looked at the second guy's point about the money:

    "Who is paying for all of that bandwidth?"

Then I realized his premise was flawed because it assumed hackers buy their own high-speed data pipelines from ISPs. 

They don't pay for anything. 

A botnet is made of thousands of compromised everyday devices—like smart TVs, home routers, and unpatched PCs owned by normal people. The hacker injects a silent script into those devices, and those devices scan the internet in the background. 

The innocent subscribers are the ones unknowingly paying the ISP bills for the electrical power and data usage. The hacker gets a global scanning network for zero cost.

Then I looked at the first guy's claim about the 90% traffic figure.

At first it sounded like pure paranoia, but then I realized there was a huge mix-up between two completely different concepts:

    Bandwidth Volume (Size of Data) vs Packet Count (Number of Requests)

If you think about **Bandwidth Volume**, humans consume almost all of it. When someone downloads a huge video file or streams a movie over an FTTH line, they use massive chunks of data. Bots don’t do that; they just send tiny, lightweight text strings.

But if you think about **Packet Count**, the first guy is actually right. 

In terms of individual, unsolicited connection requests hitting a random public IP address, the automated traffic is non-stop. Nearly half of global web traffic is automated, and the automated "knocks on the door" happen 24/7.

That eventually brought me right back to why ISPs set up physical fiber networks the way they do.

I started tracing exactly what happens when fiber enters a house:

The glass cable connects to an ONT (Optical Network Terminal). The ONT just translates light pulses into standard electrical Ethernet signals.

If you take a raw Ethernet cable out of that ONT and plug it straight into a PC without a software firewall, your computer gets a raw Public IP Address exposed to the world. 

Because botnets are constantly cycling through all 4 billion IPv4 addresses looking for open ports, they will target your machine within minutes. If your OS has an unpatched security hole, you really can get compromised almost instantly.

But in real life, ISPs protect you from this immediately using architecture. 

Right after the ONT, they install a Residential Gateway which is just a Router with a built-in hardware firewall. 

The router uses NAT (Network Address Translation). It maps all your home devices to hidden, private local IP addresses like `192.168.1.x` and automatically drops any unsolicited incoming packets from the outside world before they ever reach your computer.

To see what my own machine was doing, I opened a terminal and ran:

    netstat -abno

I saw a bunch of external connections held by a process called `zen.exe`. 

But it was literally just my open privacy browser tabs talking securely over port 443 to Google and Cloudflare servers. 

Everything else was completely internal system processes—like my graphics card drivers talking to the local loopback address `127.0.0.1`, or the core Windows network kernel keeping system ports open safely inside my own local network where external bots can't reach them.

The escalation of my mind during this session was hilarious:

    learning how light stays inside glass fiber cables
                           ↓
    checking how ISPs route fiber lines into a house
                           ↓
    finding a Reddit thread claiming you get hacked in 45 seconds
                           ↓
    worrying that my own PC was a zombie node for a botnet

It turns out that while fiber optics is a beautifully engineered physical layer of light, the data layer on top of it is just a non-stop shouting match of automated scripts trying to find an open door. 

As long as you leave the router's hardware firewall in place, the chaos stays completely on the outside.

---

[← Go to Previous Archive](/Journal/Network_Speed/README.md)
---