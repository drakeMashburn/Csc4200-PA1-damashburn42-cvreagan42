# PA1 Capture Analysis

_Your capture, `evidence/pa1-capture.pcap`, holds two ETP sessions from your own
programs: a PUT, then a GET of the same file. This analysis is about the **GET
session** unless a question says otherwise. Use Wireshark (or tshark) to read the
capture. The numbers must come from your capture: they are checked against it._

## How I captured it

_Paste the exact commands you ran (capture, server, client, ss), and the size of
the file you transferred._

```
```

## Facts

_Fill in every value from **your** capture, for the **GET** session. Integers in
decimal. Sequence numbers are the **absolute** (raw) values — in Wireshark, turn
off "relative sequence numbers" (Edit > Preferences > Protocols > TCP) or read
the "Sequence Number (raw)" field. With tshark, add
`-o tcp.relative_sequence_numbers:FALSE`. Do not change the key names or the
`etp-facts` label: a script reads this block._

```etp-facts
server_port: 47856
client_port: 5000
client_isn: 674773349                      # raw sequence number in the client's SYN
server_isn: 1056897633                     # raw sequence number in the server's SYN/ACK
client_mss: 65495                     # MSS option value in the client's SYN, in bytes
server_to_client_payload_bytes: 262156   # all TCP payload bytes the server sent in this session (HELLO_OK + GET_OK + BYE_OK)
server_to_client_data_segments: 4  # server-to-client frames with payload, across all three responses, counted as captured
first_fin_from: server                # server or client
```

## Q1. The handshake

_Give the frame number, flags, and raw sequence and acknowledgment numbers of the
three handshake segments of the GET session. Explain why the acknowledgment number
in the third segment is what it is. What MSS did the client advertise, and what on
your machine determines that value?_

- SYN
* Frame Number: 29
* Flags: SYN 0x002
* Sequence Number: 1028447834
* Acknowledgement Number: 0

- SYN-ACK
* Frame Number: 30
* Flags: SYN, ACK 0x012
* Sequence Number: 3366125845
* Acknowledgement Number: 1028447835

- ACK
* Frame Number: 31
* Flags: ACK 0x010
* Sequence Number: 1028447835
* Acknowledgment Number: 3366125846

The acknowledgment number in the third segment is one greater than the server's initial sequence number because the server's SYN flag consumes a sequence number.

The client advertised an MSS of 65,495 bytes and it is determined by the operating system's TCP/IP stack and is usually based on the network interface MTU. The MSS represents the largest TCP payload that can fit in a segment without fragmentation.

## Q2. Your protocol on the wire

_Find the segment that carries your client's GET request. Give its frame number,
paste the first 8 payload bytes in hex, and decode each header field (VERSION,
TYPE, STATUS, RESERVED, LENGTH). Do the same for the header of the server's
GET_OK. Does GET_OK's LENGTH equal the size of your file? Show the arithmetic that
connects `server_to_client_payload_bytes` to the three responses the server sent in
this session._

- GET
* Frame Number: 36
* Payload Bytes (Hex): 01 03 00 00 00 00 00 09
* Decoded Fields: VERSION: 1, TYPE: GET, STATUS: 0, RESERVED: 0, LENGTH: 9

- GET_OK
* Frame Number: 37
* Payload Bytes (Hex): 01 83 00 00 00 04 00 00
* Decoded Fields: VERSION: 1, TYPE: GET_OK, STATUS: 0, RESERVED: 0, LENGTH: 262144

After performing ls -l photo.bin in the terminal, it shows a size of 262144, matching the LENGTH of GET_OK.

HELLO_OK: 01 81 00 00 | 00 00 00 0c = 12
GET_OK: 01 83 00 00 | 00 04 00 00 = 262144
BYE_OK: 01 85 00 00 | 00 00 00 00 = 0

12 + 262144 + 0 = 262156 payload bytes

## Q3. A byte stream, not messages

_How many segments carried the **GET_OK response** (not counting the segments
that carried HELLO_OK and BYE_OK), and what were their payload sizes? Were they
all the same size? Were they all the MSS? Did any one segment carry the
end of one ETP message and the start of the next, or did one ETP message span
several segments? Explain what this means for code that reads an ETP message, and
point to the place in your own code that deals with it._

There are 7 segments that carry the GET_OK response.
* Frame 37: 32768 bytes
* Frame 38: 32768 bytes
* Frame 40: 61440 bytes
* Frame 42: 47360 bytes
* Frame 43: 22272 bytes
* Frame 45: 47360 bytes
* Frame 46: 18184 bytes

No, the segments were not all MSS-sized. Some segments were near the MSS, while others were smaller because they carried the remainder of the GET_OK payload.

No, none of the segments carried the end of one TCP message and the beginning of another. Instead, one ETP message spanned
several TCP segments. Frames 38, 40, 45, and 46 contains only continuation file data, not new ETP headers.

For the code, TCP is a byte-stream protocol, not a message protocol. One ETP message may arrive across several TCP
segments as the GET_OK message indicated. The code must read the ETP header first, extract the LENGTH field, and keep
reading until exactly length bytes have been received. The read_message() function handles this by calling
recv_exact() to read first the fixed-size ETP header and then exactly the number of payload bytes specified by the LENGTH field.

## Q4. Teardown and TIME-WAIT

_Which side sent the first FIN in the GET session, and why — cite the rule in the
specification. Paste the relevant line(s) from `evidence/time-wait.txt`. Which end
of the connection (address and port) is in TIME-WAIT, why that end, and roughly how
long Linux keeps it there? Connect this to the SO_REUSEADDR requirement._

The sever sent the first FIN because according to the specification, the client ends the ETP session by sending BYE, and the server responds with BYE_OK. After sending BYE_OK, the server initiates TCP connection teardown and therefore sends the first FIN.

Line from `evidence/time-wait.txt`: TIME-WAIT 0  0 127.0.0.1:5000 127.0.0.1:46514

The endpoint in the output shows 127:0.0.1:5000, which is the server. The server entered TIME-WAIT because it initiated TCP connection teardown by sending the first FIN. Linux keeps TIME-WAIT there for about 60 seconds. IME-WAIT prevents delayed packets from an old connection from being confused with packets belonging to a newer connection. Because recently closed connections may leave port 5000 in TIME-WAIT, the server uses SO_REUSEADDR to allow rebinding to the same port immediately after restart.

## Q5. What an observer learns

_Suppose someone else obtained this capture file. List specifically what they
learn about your session (names, sizes, timing, anything else). You captured with a
200-byte snap length: what does that hide, and what does it not hide? Which kind of
protection would stop a real on-path observer from learning these things, and what
would it still not hide?_

That person could learn your name, server name, filename, file size, TCP ports used, IP addresses,
protocol operations, sequence numbers, acknowledgment numbers, MSS values, and TCP teardown info.

It hides most of the contents of photo.bin, large portions of the GET_OK payload, and any data beyond the first 200 bytes of a packet. It does not hide the client name, server name, filename, filesize, GET/PUT/BYE/HELLO message types, source and destination points, TCP sequence and acknowledgement numbers, and timing information.

Encryption would be the stopper for a on-path observer from reading the ETP payloads and protocol contents. An observer could still see source and destination IP addresses, source and destination ports, packet timing, approximate message types, connection and start times, and total amount of data stored.


