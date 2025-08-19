---
title: Why does a magic packet repeat its MAC address 16 times?
date: 2025-08-17T12:25:50.063Z
draft: false
tags:
  - WOL
  - WakeOnLan
  - MagicPacket
  - Network
  - Ethernet
  - Hardware
category: network
lastmod: 2025-08-19T16:57:51+09:00
layout: 
Banner: 
---
I recently built a proxmox server on an unused computer. While I was having fun building my home server, I thought to myself, "I can turn off a computer remotely with ssh and use things like `shutdown`, `halt`, and `poweroff`, but can I turn on a computer that is off remotely?" I thought to myself.

I found out that it can be implemented through a technology called Wake-on-LAN (WoL), and while researching, I found something very curious, so I'm going to write a short blog post about it.

WoL is a technology that sends a signal over the network to wake up a computer in a low-power state. And what makes this possible is a piece of data called a "magic packet," which, if you look at the structure of the magic packet, sends the MAC address, which is the unique address of the computer, exactly the same 16 times. I'm curious about how this mysterious number of 16 (instead of 4 or 8) came to be, so I'm going to dig into it.

## WoL and magic packets

First, let's quickly review what Wake on LAN (WoL) and Magic Packets are.

- Wake on LAN (WoL): WoL is the protocol name given to the Magic Packet technology.  It is used to remotely wake up a powered off or sleeping PC over the network.

- Magic Packet: An Ethernet frame with a special data pattern. This packet is detected by the receiving Ethernet controller and is responsible for waking up the computer.

## Structure of a magic packet (Ethernet frame)

![[wake-on-lan-mac-address-16-times-1755498497920.webp]]

As mentioned earlier, a magic packet is an Ethernet frame with a specialized data pattern. To understand the structure of a magic packet, we need to know about Ethernet frames.

An Ethernet frame is defined by a total of five fields.

1. MAC dest: Destination MAC address, which means the physical address of the network device that will receive this packet. Usually, the broadcast address `FF:FF:FF:FF:FF:FF:FF:FF` is used to signal all devices in the network. 2.
MAC src: Source MAC address. The MAC address of the device sending this packet.
3. MISC (Misc., etc.): This is the area containing the EtherType, which indicates the type of Ethernet frame. It serves to indicate what kind of data this packet is.
4. Data / Payload: This is the part that contains the actual data to be transmitted. It includes headers and data from higher layers (e.g. TCP/IP). If the size of the data is smaller than 46 bytes, padding is added to meet the minimum size.
5. CRC (Cyclic Redundancy Check): A sequence of frame checks to detect errors that may occur during transmission. It checks for integrity similar to a hash algorithm, but more on that later.

> [!note]+ padding of Ethernet frames
> In the early days of Ethernet, all computers shared a single communication line and communicated according to a convention called CSMA/CD. CD is Collision Detection, and if two computers send data at the same time, causing a 'collision', the sender detects this collision, stops sending data, and resends it.
> The reason why the data in an Ethernet frame is smaller than 46 bytes and needs to be filled with 46 bytes is because of Collision Detection. If the data packet was too small, a data collision on the network could occur and the sender would not notice it and not resend it.
> If A sends a packet that is too small and ends the transmission in a split second, A doesn't realize that the data is broken and the data is lost.
> To ensure that collisions could still be detected in this scenario, the engineers decided to arbitrarily increase the length of the data by adding padding. The defined size was 64 bytes, and the minimum size of the actual data (64 - 18) was calculated to be 46 bytes, excluding 18 bytes of basic information in the frame.

### Magic Pattern

The **MAGIC Pattern** shown in the figure will be placed in the Payload field. The magic pattern consists of 102 bytes in total and can be divided into two parts.

- FF ... FF (6 bytes): This is called the Synchronization Stream and contains six consecutive `FF` values, which helps the Ethernet controller on the receiving end to easily scan and detect the magic packet.
- MAC 6 \* 16 (96bytes): **This is where we repeat the MAC address we want to wake up 16 times. This is the core of the magic pattern.

Additionally, Secure On Password allows you to set a password of 6 bytes in size, which is used by some WoL technologies to increase security. It is only used when the feature is enabled and prevents unauthorized users from remotely turning on the computer.

> [!note]+ How synchronization streams make detection easier.
> Synchronization streams make the scanning state machine much simpler. The scanning state machine refers to the hardware logic inside the Ethernet controller that analyzes incoming data to look for specific patterns.
> If there were no synchronization stream, the Ethernet controller would have to keep checking every incoming byte sequence to see when the pattern of a 6-byte MAC address repeated 16 times would start. This becomes a more complex and power hungry operation.
> But if a clear and distinctive synchronization stream called FF ... FF comes first, the controller can start scanning for subsequent MAC address patterns only when it detects this six-byte sequence. In other words, the synchronization stream acts as a kind of beacon or marker, helping the controller minimize unnecessary scanning and focus on finding patterns only when it needs to.

Now that we've rambled on for a bit, let's take a look at why MAC addresses are sent 16 times in a row.

## Why repeat the MAC address 16 times?


I searched for an answer to this question in Korean articles, but there was no answer except "This is what happens when you send a packet like this!" and even when I asked Gemini, they gave me the following strange answer.

For reference, the question was: **Why does the magic packet repeat the MAC address 16 times instead of just another number?

![[remote-wakeonlan-1755481071615.webp]]
~~clap~!

I came across a 2014 article on a site called serverfault that kind of answered the question. It's not an official document, and it's not accurate, but the logic of the respondent convinced me, so I pulled it up.

```embed
title: "Why does a Wake-On-LAN packet contain 16 duplications of the target MAC address?"
image: "https://cdn.sstatic.net/Sites/serverfault/Img/apple-touch-icon@2.png?v=9b1f48ae296b"
description: "From the Wireshark webpage: The Target MAC block contains 16 duplications of the IEEE address of the target, with no breaks or interruptions. Is there any specific reason for the 16 duplications?"
url: "https://serverfault.com/questions/632908/why-does-a-wake-on-lan-packet-contain-16-duplications-of-the-target-mac-address"
```

The key is simplicity and reliability of the implementation.

When AMD and Hewltt Packard (HP) first created Magic Packet, their goal was to add the feature as little as possible to existing hardware configurations, so they decided to utilize the address matching circuitry already present in Ethernet controllers to detect Magic Packets.
The address matching circuitry is responsible for identifying and receiving only the frames that match the MAC address of the PC in question out of the many incoming Ethernet frames on the network. To add the ability to detect magic packets, we simply added a counter that repeatedly counts the MAC address 16 times.

We use a 4-bit counter that can hold values from 0 to 15, which allows for simple logic to execute wakeup code when the counter overflows (15+1=0). According to the pseudo code in the link above, it looks like this

```text
Initialize a single counter that holds values from 0-15.
Set it to 0.
Watch the network. When I see the signal:
Loop:
  Do I see my address at the right spot?
  Yes: Add 1 to counter.
    Did I just overflow? (15+1 = 0?)
    Yes: Jump out of loop to "wake up" code.
...otherwise
Loop again.
```

This is all about the simplicity of the implementation, and the number 16 comes from the reliability of the implementation. As mentioned earlier, the Internet is a fast-paced environment with lots of information flowing back and forth. To prevent a random stream of data from becoming a command to turn on a system, it needs to be easily identifiable and have little chance of false positives.

### What if we use a 2-bit counter?

If we were to use a 2-bit counter and repeat the MAC address only 4 times, it would be 24 bytes, which is less than the minimum size of frame data (46 bytes), so we would add padding. This would make it much more likely that such a sequence would occur on the network by chance, greatly increasing the chance of unintentionally waking up the system. It would also have weakened the characteristics that distinguish magic packets from other network traffic, making them less reliable.
To summarize, using a 2-bit counter to iterate the MAC address only 4 times would have maintained the simplicity of the hardware implementation, but would have been significantly worse than the current 16 iterations in terms of "misbehavior prevention" and "high reliability", two of the most important goals.

### What if we use 8-bit counters?

An 8-bit counter can represent values from 0 to 255, and in order for the counter to overflow (255+1=0), the MAC address must be repeated 256 times. Since a MAC address is 6 bytes, 256 iterations would be **1,536 bytes**. This exceeds the maximum size of an Ethernet frame, so it is not feasible in practice, and even if it were, the frame would be unnecessarily large and we would have to move a lot of data around pointlessly.

### 4-bit counter

So why a 4-bit counter? Let's put the previous hypotheses together

- A 2-bit counter (iterated 4 times) is too risky ** The payload size is too small, increasing the chance of false positives. There was a risk that data on the network could be accidentally recognized as a magic packet, causing the system to turn on unwantedly. **Failed on reliability grounds.
- The payload size exceeds the Ethernet maximum transmission unit (MTU), making it impossible to send over standard networks in the first place. Rejected for **practicality**.

In the end, a 4-bit counter (16 iterations) was the sweet spot that solved both problems. It is long enough (96 bytes) to maximize reliability, while remaining fully compliant with the Ethernet frame standard and not losing practicality. At the same time, it minimized the complexity of the hardware, as it could be implemented as a simple 4-bit counter.

Within these constraints, 16 was the number that captured the best of both worlds: reliability, efficiency, and simplicity.
## Conclusion

What started as a curiosity about the number 16 while simply configuring a home server has led me to this point. With the limited resources of early Ethernet controllers, minimum frame sizes, and the need to distinguish between random data and signals, engineers found a solution.
The fact that such a seemingly trivial technical detail has such a rational reason and solution is one of the true fascinations of engineering.