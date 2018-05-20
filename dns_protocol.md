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

## The C/S rules

* If send a Request from Client, only `Header+Question(s)` in the package.
* If send a Response from Server, will have all in package.
