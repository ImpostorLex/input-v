---
{"dg-publish":true,"permalink":"/cards/networking/the-threeway-handshake/"}
---

~ [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]] | ~ [[atlas/networking\|networking]]
### Introduction
---
The threeway handshake is initiated everytime an application or any type of computer network communication as long as they are using [[TCP\|TCP]] protocol.

![The threeway handshake.png|350](/img/user/cards/networking/images/The%20threeway%20handshake.png)

**Step 1:**

**SYN** is used to iniate the connection and synchronise both devices.

**Step 2:**

SYN/ACK is sent back(if the other devices agrees to the connection) to acknowledge the synch attempt.

**Step 3:**

ACK is used by client or server to acknowledge if the series of packet have been succesfully received.

**Step 4:**

DATA is used when an connection is established and data such as bytes in a file is sent via the DATA message.

**Step 5:**

FIN properly close the connection.

**Step #:**

RST if both one of the client or server has an error will be used as a last resort.

## Sequence of Number

Any data being sent is given a random number sequence and it is use to reconstruct the sent data, data given random number is incremented by 1 per piece of data.

1.  **SYN** - Client: Here's my Initial Sequence Number(ISN) to SYNchronise with (0)
2.  **SYN/ACK** - Server: Here's my Initial Sequence Number (ISN) to SYNchronise with (5,000), and I ACKnowledge your initial number sequence (0)
3.  **ACK** - Client: I ACKnowledge your Initial Sequence Number (ISN) of (5,000), here is some data that is my ISN+1 (5,000 + 1)

## TCP close connection

![The threeway handshake-1.png|350](/img/user/cards/networking/images/The%20threeway%20handshake-1.png)

The best practice is to save as much as possible of system resource that is why after succesful transaction of data the TCP connection is quickly closed.

A device will send ACK and FIN packet to the other device and the other device will and should response with an ACK.
