# DNS protocol

## The UDP Package

```text
                               +--> # Header: must have a header
  DNS(UDP Package)             |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15|
+------------------+           |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+                                  {  QR: (1 bit) 0=REQ / 1=RES;
|      Header      |  ---------+   |                               ID                              | --> 16bit, Auto ID from Sender   {  OPCODE: (4 bit) 0=default / 1=trackback / 2=SRVstatus / 3~15=NONE;
+------------------+               +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+                                  {  AA: (1 bit) Authoritative Answer (bool=0x00|0x01);
|    Question(s)   |  ---------+   | QR|     OPCODE    | AA| TC| RD| RA|     Z     |     RCODE     | -------------------------------> {  TC: (1 bit) TrunCation (bool=0x00|0x01);
+------------------+           |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+                                  {  RD: (1 bit) Recursion Desired (bool=0x00|0x01);
|   Answer (RRs)   |  -----+   |   |                            QDCOUNT                            | --> 16 bit, RRs in Question(s)   {  RA: (1 bit) Recursion Available (bool=0x00|0x01)
+------------------+       |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+                                  {  Z: (3 bit) NONE (use 0x00,0x00,0x00)
| Autherity (RRs)  |  -----+   |   |                            ANCOUNT                            | --> 16 bit, RRs in Answer        {  RCODE: (4 bit) { 0=No_error_condition / 1=Format_error / 2=Server_failure
+------------------+       |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+                                                    { 3=Name_Error / 4=Not_Implemented / 5=Refused / 6~15=NONE
| Additional (RRs) |  -----+   |   |                            NSCOUNT                            | --> 16 bit, RRs in Authority
+------------------+       |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |   |   |                            ARCOUNT                            | --> 16 bit, RRs in Additional
                           |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |   |
                           |   |
                           |   +--> # Question(s): every one of data need include this
                           |       | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15|
                           |       +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |       |                                                               |     { Unkown length, end with "0x00".
                           |       :                             QNAME...                          : --> { split with "." and join a length of every value at first.
                           |       :                                                               :     { E.g. (deepwn.com) => 0x06,('deepwn' to hex),0x03,('com' to hex),0x00
                           |       +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |   +---|                             QTYPE                             | --> 16 bit, RRs type
                           |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |   |   |                             QCLASS                            | --> 16 bit, RRs class
                           |   |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                           |   |
                           |   |
                           +------> # Resource Records (If is a Question, it can without this part)
                               |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15|
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |   |                                                               |
                               |   :                              NAME...                          : --> Unkown length, NAME must same like QNAME
                               |   :                                                               :
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               +---|                              TYPE                             | --> 16 bit, Same like the QTYPE
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |   |                              CLASS                            | --> 16 bit, Same with QCLASS
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |   |                               TTL                             | --> 32 bit, for Time live
                               |   |                                                               |
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |   |                            RDLENGTH                           | --> 16 bit, RDATA length
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |   |                                                               |
                               |   :                             RDATA...                          : --> Unkown length, the data
                               |   :                                                               :
                               |   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                               |
                               |
                               +--> # Types
                                   +-----------+-------+-------------------------------------------+
                                   |  (Q)TYPE  | Value |                 Meaning                   |
                                   +-----------+-------+-------------------------------------------+
                                   |     A     |   1   |            a host address                 |
                                   |-----------|-------|-------------------------------------------|
                                   |     NS    |   2   |       an authoritative name server        |
                                   |-----------|-------|-------------------------------------------|
                                   |     MD    |   3   |  a mail destination (Obsolete - use MX)   |
                                   |-----------|-------|-------------------------------------------|
                                   |     MF    |   4   |   a mail forwarder (Obsolete - use MX)    |
                                   |-----------|-------|-------------------------------------------|
                                   |   CNAME   |   5   |      the canonical name for an alias      |
                                   |-----------|-------|-------------------------------------------|
                                   |    SOA    |   6   |  marks the start of a zone of authority   |
                                   |-----------|-------|-------------------------------------------|
                                   |     MB    |   7   |   a mailbox domain name (EXPERIMENTAL)    |
                                   |-----------|-------|-------------------------------------------|
                                   |     MG    |   8   |    a mail group member (EXPERIMENTAL)     |
                                   |-----------|-------|-------------------------------------------|
                                   |     MR    |   9   | a mail rename domain name (EXPERIMENTAL)  |
                                   |-----------|-------|-------------------------------------------|
                                   |    NULL   |   10  |        a null RR (EXPERIMENTAL)           |
                                   |-----------|-------|-------------------------------------------|
                                   |    WKS    |   11  |    a well known service description       |
                                   |-----------|-------|-------------------------------------------|
                                   |    PTR    |   12  |          a domain name pointer            |
                                   |-----------|-------|-------------------------------------------|
                                   |   HINFO   |   13  |             host information              |
                                   |-----------|-------|-------------------------------------------|
                                   |   MINFO   |   14  |     mailbox or mail list information      |
                                   |-----------|-------|-------------------------------------------|
                                   |     MX    |   15  |               mail exchange               |
                                   |-----------|-------|-------------------------------------------|
                                   |    TXT    |   16  |               text strings                |
                                   +-----------+-------+-------------------------------------------+

```

**A Package must & maybe only have `Header+Question(s)` in the package.**

## Example

Request:

```text
0000   44 a5 01 00 00 01 00 00 00 00 00 00 04 74 65 73   D¥...........tes
0010   74 06 64 65 65 70 77 6e 03 63 6f 6d 00 00 01 00   t.deepwn.com....
0020   01                                                .


                | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x44a5                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    | 0 | 0   0   0   0 | 0 | 0 | 1 | 0 | 0   0   0 | 0   0   0   0 | ( = 0x0100 )
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
        Header  |                             0x0001                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0000                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0000                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0000                            |
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    : 0x04746573740664656570776e03636f6d00 (04test06deepwn03com00)  :
           |    :         (No 0x00 fixed, just one 0x00 at end of all)          :
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       Queries  |                           0x0001 (A)                          |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                          0x0001 (inet)                        |
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

    Domain Name System (query)
    Transaction ID: 0x44a5
    Flags: 0x0100 Standard query
        0... .... .... .... = Response: Message is a query
        .000 0... .... .... = Opcode: Standard query (0)
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... .0.. .... = Z: reserved (0)
        .... .... ...0 .... = Non-authenticated data: Unacceptable
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 0
    Queries:
        test.deepwn.com: type A, class IN
            Name: test.deepwn.com
            [Name Length: 15]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)

```

Response:

```text
0000   44 a5 81 83 00 01 00 00 00 01 00 00 04 74 65 73   D¥...........tes
0010   74 06 64 65 65 70 77 6e 03 63 6f 6d 00 00 01 00   t.deepwn.com....
0020   01 c0 11 00 06 00 01 00 00 01 4f 00 37 04 76 69   .À........O.7.vi
0030   70 31 06 61 6c 69 64 6e 73 c0 18 0a 68 6f 73 74   p1.alidnsÀ..host
0040   6d 61 73 74 65 72 07 68 69 63 68 69 6e 61 c0 18   master.hichinaÀ.
0050   78 3a 0d b2 00 00 0e 10 00 00 04 b0 00 00 0e 10   x:.².......°....
0060   00 00 01 68                                       ...h


                | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x44a5                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    | 1 | 0   0   0   0 | 0 | 0 | 1 | 1 | 0   0   0 | 0   0   1   1 | ( = 0x8183 )
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
        Header  |                             0x0001                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0000                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0001                            |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                             0x0000                            |
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    : 0x04746573740664656570776e03636f6d00 (04test06deepwn03com00)  :
           |    :                 (just add 0x00 at end of name)                :
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       Queries  |                           0x0001 (A)                          |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                          0x0001 (inet)                        |
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |      0xc011 (pin tag "0xc0"; fixed 17 byte from Queries)      |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     Autherity  |                          0x0006 (SOA)                         |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                          0x0001 (inet)                        |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                      0x0000014f (TTL: 355)                    |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |                     0x0037 (RData Length: 55)                 |
           |    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
           |    |            RDATA (NameServer & mail & blablabla...            |
           *->  +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

    Domain Name System (response)
    Transaction ID: 0x44a5
    Flags: 0x8183 Standard query response, No such name
        1... .... .... .... = Response: Message is a response
        .000 0... .... .... = Opcode: Standard query (0)
        .... .0.. .... .... = Authoritative: Server is not an authority for domain
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... 1... .... = Recursion available: Server can do recursive queries
        .... .... .0.. .... = Z: reserved (0)
        .... .... ..0. .... = Answer authenticated: Answer/authority portion was not authenticated by the server
        .... .... ...0 .... = Non-authenticated data: Unacceptable
        .... .... .... 0011 = Reply code: No such name (3)
    Questions: 1
    Answer RRs: 0
    Authority RRs: 1
    Additional RRs: 0
    Queries:
        test.deepwn.com: type A, class IN
            Name: test.deepwn.com
            [Name Length: 15]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Authoritative nameservers:
        deepwn.com: type SOA, class IN, mname vip1.alidns.com
                Name: deepwn.com
                Type: SOA (Start Of a zone of Authority) (6)
                Class: IN (0x0001)
                Time to live: 335
                Data length: 55
                Primary name server: vip1.alidns.com
                Responsible authority's mailbox: hostmaster.hichina.com
                Serial Number: 2017070514
                Refresh Interval: 3600 (1 hour)
                Retry Interval: 1200 (20 minutes)
                Expire limit: 3600 (1 hour)
                Minimum TTL: 360 (6 minutes)

```
