# Task 1 - SMTP Protocol Analysis

Wireshark capture analysis of a complete SMTP email session with a file attachment. Walks through every stage from the TCP handshake to session close.

## Capture File

`mail_sender_attachment.pcapng` - open in Wireshark and follow the TCP stream to see the full exchange.

## What SMTP Looks Like on the Wire

SMTP is a push protocol. The client pushes the message outward to the server, step by step, using a command-response pattern. It runs on port 25 (server to server) or port 587 (client to server).

### TCP Handshake

Before any mail is sent, TCP sets up the connection:

```
Client  ->  Server   SYN
Server  ->  Client   SYN-ACK
Client  ->  Server   ACK
```

### SMTP Command Sequence

| Step | Command / Response | What it does |
|------|--------------------|--------------|
| 1 | 220 | Server ready greeting |
| 2 | EHLO | Client identifies itself, requests ESMTP extensions |
| 3 | 250 (multi-line) | Server lists supported capabilities |
| 4 | MAIL FROM | Sender address declared |
| 5 | RCPT TO | Recipient address declared |
| 6 | DATA | Start of message content |
| 7 | 354 | Server says go ahead |
| 8 | headers + body | Full message including headers, then body, then a single `.` to end |
| 9 | 250 | Server confirms message received |
| 10 | QUIT | Client closes session |
| 11 | 221 | Server acknowledges, closes connection |

### Attachment

The attachment is embedded in the DATA section as Base64. Since there is no TLS on this session it shows up in the capture as plain text. You can copy the Base64 block and decode it manually to get the original file back.

To follow the stream in Wireshark: right-click any SMTP packet, Follow, TCP Stream.

### Session Close

```
Client  ->  Server   QUIT
Server  ->  Client   221 Bye
Client  ->  Server   FIN
Server  ->  Client   FIN-ACK
```

## Security Note

This session has no encryption. The sender address, recipient, subject, message body, and attachment are all readable in the capture. In any real environment you would add STARTTLS or use port 465 with SMTPS to wrap the session in TLS before any of that data moves.
