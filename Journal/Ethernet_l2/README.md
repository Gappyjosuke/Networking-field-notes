[← Back to Networking Archive](/README.md)

# Can Two Computers Communicate Without IP Addresses?
### Breaking Down A Layer 2 Networking Question Through Experimentation

---

## The Question That Started Everything

This entire thing started from a simple question while working with Cisco Packet Tracer.

Whenever networking i usually connect two computers together, the first thing we usually do is assign IP addresses.

>For example:

<div align="center">
    
| Device | IP Address |
|----------|----------|
| PC1 | `192.168.1.1` |
| PC2 | `192.168.1.2` |

</div>

Once the addresses are assigned correctly, communication works.

```bash
ping 192.168.1.2
```

returns replies and everything appears normal.

However, if the IP addresses are removed, communication appears to stop immediately.

After seeing this behavior repeatedly, it becomes very easy to develop the following model:

<div align="center">

```text
No IP Address → No Communication
````
</div>

At first glance this conclusion seems completely reasonable.

However, something about it felt incomplete.

Because every Ethernet network card already possesses an address of its own.

>For example:
<div align="center">
    
| Device | MAC Address |
|----------|----------|
| PC1 | `00:0c:29:4e:a4:ef` |
| PC2 | `00:0c:29:a5:60:14` |

</div>

This immediately raises a deeper question.

If Ethernet devices already have unique addresses, then why are IP addresses necessary for two directly connected computers to communicate?

Or phrased more precisely:

> Can Ethernet itself transport data between two computers using only MAC addresses without involving IP at all?

Before answering that question, it is important to identify exactly what is failing when communication stops.

---

## Why Ping Is Not Actually Testing Ethernet

Most of us usually, tend to use ping as a generic network test.

The logic usually looks something like this:

<div align="center">

```text
Ping Works  → Network Works

Ping Fails  → Network Broken
```

</div>

The problem is that ping is not actually an Ethernet protocol.

like we already know Ping uses ICMP, 
ICMP depends on IP.

IP itself depends on Ethernet.

The relationship looks roughly like:

<div align="center">

```text
Ping → ICMP → IP → Ethernet → Physical Link
```
</div>

This is an important realization we should consider because it means that a failed ping does not necessarily prove Ethernet is failing.

It only proves that something somewhere in the chain above is not functioning correctly.

So Instead of asking:

> Can two computers communicate without IP?

the better question becomes:

> Can Ethernet transport data between two machines without involving IP at all?

Now the problem is much more specific and much easier to test.

---

## Looking At What Ethernet Actually Contains

To answer the question, it helps to examine what an Ethernet frame actually looks like.

A simplified Ethernet frame contains the following information:

<div align="center">
    
```text
+--------------------+
| Destination MAC    |
+--------------------+
| Source MAC         |
+--------------------+
| EtherType          |
+--------------------+
| Payload            |
+--------------------+
```
</div>

One thing immediately stands out.
There is no IP address present here.
Ethernet fundamentally operates using MAC addresses.

>For example from my VM's:

<div align="center">

| Field | Example Value |
|---------|---------|
| Destination MAC | `00:0c:29:a5:60:14` |
| Source MAC | `00:0c:29:4e:a4:ef` |

</div>

This observation naturally suggests an experiment.

Instead of allowing the operating system to create an IP packet and then wrap it inside an Ethernet frame, what if we manually create the Ethernet frame ourselves and place our own data directly inside it?

Conceptually the experiment would look like this:

<div align="center">

```text
Custom Ethernet Frame → Network → Destination Computer
```

</div>

If the destination machine receives the frame, then we have direct answer that Ethernet communication is possible without IP basically that how earlier times system are buit like. Each vendor with thier own way of communication between thier systems.

---

## Designing The Experiment

To test the idea, I used two virtual machines connected through VMware.

The setup looked like this:

<div align="center">

Rocky Linux → VMware Virtual Network → Windows 11

</div>

The machines had the following MAC addresses:

<div align="center">

| Device | MAC Address |
|----------|----------|
| Rocky linux server | `00:0c:29:4e:a4:ef` |
| Windows 11 | `00:0c:29:a5:60:14` |

</div>

The goal was straightforward.

I wanted my Rocky linux server to send an Ethernet frame directly to the Windows machine.

Not an IP packet.
Not a TCP packet.
Not a UDP packet.
Not an ICMP echo request.

Instead with a raw Ethernet frame carrying data.

---

## The Next Problem

At this point another question appeared.

Even if Ethernet supports this type of communication, how can a normal application actually create an Ethernet frame?

Most applications never interact with Ethernet directly.

A normal application typically relies on the operating system networking stack.

The process usually looks like:

<div align="center">

```text
Application → TCP/UDP → IP → Ethernet → Network Interface
```
</div>

The lower layers are normally hidden from the programmer.

Therefore we need a way to manually construct Ethernet frames ourselves.

---

## Using Scapy To Construct Ethernet Frames

Scapy is a packet manipulation library for Python that allows network protocols to be constructed programmatically.

Instead of manually writing raw bytes, we can describe the packet structure using Python objects.

>For example:

```python
Ether(
    dst="00:0c:29:a5:60:14"
)
```

can be interpreted as:

```text
Create an Ethernet header.

Destination MAC:
00:0c:29:a5:60:14
```

Notice that nothing is being transmitted yet.

We are simply describing what the Ethernet frame should contain.

---

## Understanding Each Field Before Writing Code

Before constructing the frame, it is worth understanding each field individually.

The first field is:

```python
dst="00:0c:29:a5:60:14"
```

This specifies the destination MAC address.

In other words:

```text
Who should receive this frame?
```

For our experiment, the answer is:

```text
Windows 11 VM
```

The next field is:

```python
type=0x1234
```

This field is known as the EtherType.

EtherType tells the receiving machine what kind of payload is contained inside the Ethernet frame.

Common EtherTypes include:

<div align ="center">
    
| EtherType | Meaning |
|------------|------------|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |

</div>

For this experiment:

```python
type=0x1234
```

was selected simply because it is easy to identify inside Wireshark.

Nothing special happens because of this value.

It merely acts as a recognizable marker for our experiment.

---

## Constructing The Payload

The payload is the actual data carried by the Ethernet frame.

For testing purposes the following message was used:

```text
Vanakam daa mapla rocky la irundhu
```

Scapy requires the payload to be represented as bytes.

Therefore:

```python
message.encode()
```

is used to convert the text into a sequence of bytes suitable for transmission.

This is necessary because Ethernet transmits bytes rather than Python strings.

---

## Building The Ethernet Frame

Once the destination MAC, EtherType, and payload are available, they can be combined into a complete Ethernet frame.

```python
from scapy.all import *

myframe = Ether(
    dst="00:0c:29:a5:60:14",
    type=0x1234
) / Raw(
    load=b"vanakkam daa mapla rocky la irundhu "
)
```

At this stage nothing has been transmitted.

The frame only exists in memory.

Think of it as filling out an envelope before placing it into a mailbox.

---

## Sending The Frame

To actually transmit the frame:

```python
sendp(
    myframe,
    iface="ens160"
)
```

is used.

Notice the function name carefully.

```python
sendp()
```

The "p" indicates Layer 2 packet transmission.

This is important because:

```python
send()
```

typically operates at Layer 3 and expects IP-based communication.

Since the entire purpose of this experiment is to bypass IP, `sendp()` is the correct choice.

---

## Verifying The Result

Sending a frame is only half the experiment.

We also need evidence that it actually arrived.

To verify the result, Wireshark was started on the Windows virtual machine.

If the frame arrived successfully, Wireshark should display:

<div align="center">

| Field | Expected Value |
|---------|---------|
| Destination MAC | `00:0c:29:a5:60:14` |
| EtherType | `0x1234` |

</div>

along with the payload:

```text
vanakkam daa mapla rocky la irundhu
```

When the frame appeared inside Wireshark, it confirmed that the transmission had succeeded.

The message had travelled from Rocky Linux to Windows.

No IP packet was involved.

No ping was used.

No transport-layer protocol participated.

The communication occurred entirely at Layer 2.

---

## What This Small Experiment Actually Proves

At this point it is important to avoid drawing the wrong conclusion.

This experiment does not prove that IP addresses are unnecessary.

IP exists to solve problems that Ethernet cannot solve efficiently.

For example:

- Communication across multiple networks
- Routing through routers
- Logical addressing
- Scalable internetworking

Ethernet solves a different problem.

Ethernet handles communication on the local network segment.

The experiment simply demonstrates that:

> Ethernet is fully capable of transporting data between local devices using only MAC addresses.

That is precisely its job.

---

## Final Model

IP is not the network itself.

IP is a layer built on top of Ethernet.

The physical medium never sees:

```text
192.168.58.129
```

Instead it sees:

```text
00:0c:29:a5:60:14
```

which is the MAC address.

Understanding that distinction was the most valuable lesson from the entire thing.

---

## Key Takeaways

- Ping doesn't mean Ethernet.
- ICMP depends on IP.
- IP depends on Ethernet.
- Ethernet fundamentally uses MAC addresses.
- Ethernet can transport data without IP.
- Wireshark can be used to observe Layer 2 communication directly.
- IP and Ethernet solve different problems.
- Understanding networking becomes much easier when each layer is studied independently.