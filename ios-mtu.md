# Reverse Enginnering reasoning for iOS BLE MTU sizes

Starting with iOS10, the ATT MTU size negotiated is 185 bytes. Prior to that it was 158 bytes. While these numbers may initially seem random, read on to understand how they are actually quite clever!

# Factors to consider when negotiating the ATT MTU size

Different BLE devices support different ATT MTU sizes. Larger MTU sizes reduce the relative packet overhead and help achieve better [throughput](https://github.com/chrisc11/ble-guides/blob/master/ble-throughput.md)

However, there are additional factors one should also take into account when negotiating an MTU size:

* **Minimizing Latency** - When using BLE, data transfers start at sync points called *connection events* The amount of data a given BLE device can send during a *connection event* is vendor specific. It's possible with a large MTU size it may take multiple connection events to transmit one MTU worth of data. Depending on the frequency of the *connection events* (7.5ms - 4s), this could mean there is a considerable lag before all the data for one MTU is received by the peer device
* **Optimal Packing** - ATT messages are traveling over *Link Layer (LL)* packets. LL packets can hold a maximum data payload of 27 bytes. Ideally, one would want to select an MTU size such that the data fits perfectly across these 27 byte data payloads. Otherwise, some throughput is being sacrificed because the final LL packet has more overhead due to the smaller data field.

# So why does iOS10 use 185 byte MTU?!

As you may recall, when transmitting an ATT MTU worth of data, there is a L2CAP header within the LL data payload that takes up 4 bytes. Thus, transferring a 185 byte ATT MTU will require 189 bytes worth of link layer payload. This means we need `189/27 = 7` LL packets. 

If you airtrace an iOS10 device, you will find the maximum number of packets sent during a *connection event* will be 7!

If you run the same analysis for the older iOS MTU size of 158, you will find that the result is exactly 6 LL packets. An airtrace will also confirm this is the maximum number of packets sent during a *connection event*


# What about Android?

Android lets a user perform an MTU exchange up to 512 bytes (even though many Android phones only support 23 byte MTUs :smile:). It's up to the developer to consider what an optimal MTU size may be for their given use case.