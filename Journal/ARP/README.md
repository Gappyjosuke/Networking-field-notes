[← Back to Archive](/README.md)

# One Random Wireshark Session Somehow Turned Into Me Learning About ARP, MAC Addresses, MITM Attacks, Fake Networks And Packet Crafting

On that day I randomly opened Wireshark while connected to my home Wi-Fi because I wanted to see what kind of packets were moving around inside my network. I didn't really have a proper goal. I was mostly just curious and wanted to observe random things and slowly understand how networking actually works underneath instead of only using the internet normally every day without thinking about what happens behind the scenes.

While looking at packets, one thing immediately caught my attention. I kept seeing the same message appear repeatedly:

    ARP Announcement for 192.168.x.x

The source attached to it was:

    SaiNXTTechno_xx:xx:xx

This instantly confused me because there is actually a nearby computer/network shop near my area called "Saixxxx.xxxx". So my brain immediately connected both things together and I started wondering if somebody nearby was somehow intercepting traffic or spying on the network.

At this point I only knew one thing:

    192.168.x.x = router/default gateway

But I didn't understand why ARP packets kept repeating constantly or why devices even needed to keep announcing things repeatedly even though they were already connected to the Wi-Fi.

So I started asking myself random questions like:

    why does this keep happening?
    why are packets being broadcasted?
    why does everything point to the router?
    do all people using the same ISP have the same gateway?
    how do devices actually know where to send packets?
    how do MAC addresses work underneath?
    why does Wireshark know device names?

Then I started checking things manually using commands like:

    ipconfig

and:

    arp -a

I started comparing the MAC address shown inside Wireshark with the MAC address stored in my ARP table. Then I realized something funny.

The suspicious device:

    xx:xx:xx:xx:xx:xx

was literally my own router.

The "SaiNXTTechno" part was simply vendor/manufacturer information attached to the MAC address. My entire suspicion basically came from the fact that the vendor name looked similar to the nearby computer shop.

After staring at ARP packets for a while, I slowly started answering the questions that originally confused me.

One thing I kept wondering was:

    why does this keep happening?

At first I thought devices would discover each other once and then never ask again. But later I learned that devices store IP → MAC mappings temporarily inside something called an ARP cache. After some time those entries expire, so devices ask again to refresh the mapping. Routers also periodically announce themselves to make sure devices still know where the gateway is. That is why Wireshark constantly shows ARP traffic even when the network looks idle.

Another thing that confused me was:

    why are packets being broadcasted?

Then I learned that ARP requests are actually broadcast packets. When a device does not know which MAC address belongs to an IP address, it sends a message to everybody on the local network basically asking:

    Who has 192.168.x.x?

That message gets sent to every device connected to the local network because the sender does not yet know which device owns the IP address.

Then the correct device replies back with its MAC address.

That also explained why Wireshark could see the packets so easily because broadcasts are visible to all devices inside the local network.

Another thing I kept thinking about was:

    why does everything point to the router?

Then I slowly understood something important about routing.

Most internet traffic is not meant for devices inside the local network. If I open YouTube or Google, my laptop cannot directly reach those servers because they are outside my local network.

So the laptop sends everything to the default gateway first:

    192.168.x.x

which is the router.

The router then forwards the traffic to the internet.

That is why the router becomes the center of almost everything happening inside the network.

I also wondered:

    do all people using the same ISP have the same gateway?

At first I thought maybe everybody using the same ISP somehow shared the same router gateway internally.

Then I realized the gateway is local to each network.

Even though many routers use addresses like:

    192.168.1.1

those are private local addresses reused independently inside millions of different homes because private IP ranges are not globally unique on the internet.

Another thing that confused me was:

    how do devices actually know where to send packets?

That eventually led me into understanding the difference between IP addresses and MAC addresses.

I slowly realized devices basically think in layers.

Applications usually think in terms of IP addresses and domains.

But Ethernet/Wi-Fi communication underneath actually works using MAC addresses.

So before packets physically travel across the local network, devices first need to discover the MAC address associated with the destination IP address.

That is exactly what ARP does.

ARP basically acts like a translator between:

    IP Address → MAC Address

inside local networks.

Another thing I found interesting was:

    how do MAC addresses work underneath?

I originally thought MAC addresses were permanent hardware identities directly burned into devices forever.

Then I learned that network cards do have factory MAC addresses, but operating systems can override the visible MAC address being used on the network.

That suddenly explained how MAC spoofing is possible.

I also learned that switches and routers use MAC addresses underneath to decide where Ethernet frames physically travel inside the local network.

Then I started wondering:

    why does Wireshark know device names?

At first I thought Wireshark somehow magically knew personal information about devices.

Then I learned that Wireshark uses public MAC vendor databases.

The first few bytes of a MAC address identify the manufacturer/vendor of the network interface.

So Wireshark simply checks the MAC prefix and translates it into vendor names like:

    Intel
    Liteon
    Hon Hai
    TP-Link

and many others.

That was the reason my router appeared with the name:

    SaiNXTTechno

instead of only showing the raw MAC address.

After understanding all of this, ARP traffic slowly stopped looking like random noise and started looking more like devices constantly asking questions and responding to each other underneath the internet.

Then the rabbit hole became much deeper.

I started wondering about public Wi-Fi networks and suddenly thought:

    if devices trust ARP replies this easily, then how do Man in the Middle attacks actually work?

At first I genuinely couldn't imagine how somebody could pretend to be a router. I kept thinking packets were some deeply protected thing only hardware could generate automatically. I couldn't understand how a random person could simply fake network traffic or impersonate another device.

Then I slowly learned something important.

Packets are basically just structured bytes.

Programs can actually build packets and Ethernet frames manually and send them through the network card. That realization completely changed how I thought about networking.

I originally imagined network cards only transmitted "real" traffic automatically generated by the operating system, but then I learned software can intentionally construct packets itself.

Then I started understanding ARP spoofing conceptually.

The attacker is not magically becoming the router. Instead they send fake ARP replies that basically tell victims:

    192.168.x.x = attacker MAC address

instead of the router's real MAC address.

Then the victim starts sending packets to the attacker machine first because the ARP cache now contains the wrong MAC mapping.

That led me into another confusion.

I started wondering:

    how can somebody change a MAC address if MAC addresses are burned into hardware?

Then I learned that even though hardware has a real factory MAC address underneath, operating systems can override the MAC address being presented to the network. That means devices can temporarily pretend to be another MAC address locally.

That suddenly explained how MAC spoofing is possible.

After understanding that, I somehow ended up going even deeper into networking discussions and random Reddit threads about fake Wi-Fi environments, rogue devices, packet captures, cloned phones, decoy networks and virtual machines.

One Reddit comment especially confused me because the person described changing their real Wi-Fi network while leaving the old network environment alive using small virtual machines and packet captures. Then they said every time a suspicious device reconnected, some kind of DoS/disruption automatically started again.

At first the entire thing sounded like nonsense to me.

I couldn't even understand what things like:

    "minimum VM's"
    "rogue cloned phone"
    "packet captures"
    "fake environment"

actually meant.

So I started trying to imagine how something like that would even work.

Then I slowly realized the person was probably not doing some hacking attack at all. They most likely created a fake/decoy network environment using lightweight virtual machines, packet captures, monitoring scripts and automation to observe reconnecting devices and suspicious traffic behavior.

The deeper I went, the more I realized networking is basically programmable logic happening underneath constantly.

Packets are just structured data.

Frames can be crafted.

MAC addresses can be overridden.

Devices trust ARP replies very easily.

Routers constantly announce themselves.

Wireshark is basically showing conversations happening invisibly all the time underneath normal internet usage.

The funniest part is how fast my brain escalated from:

    weird ARP packet

to:

    nearby shop spying on my network??

to:

    wait this is literally my own router

to:

    how do MITM attacks work internally?

to:

    how can software create packets?

to:

    how do fake network environments even work?

All of this somehow started from one repeated ARP packet inside Wireshark.


---

[← Back to Archive](/README.md)
---

[Go to Next Archive →](/Journal/DNS/README.md)
---
