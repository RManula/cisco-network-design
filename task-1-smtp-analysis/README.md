# SMTP Protocol Analysis

Analyzed a Wireshark packet capture of a complete SMTP session where a client connects to a mail server, authenticates, sends a message with an attachment, and closes the connection. The session runs without TLS so everything is visible in plain text.

Open `mail_sender_attachment.pcapng` in Wireshark. Right-click any SMTP packet, choose Follow, then TCP Stream to see the full exchange in one view.

---

## How the Session Works

### 1. TCP Handshake

Before any SMTP commands, TCP establishes the connection:

```
Client  ->  Server    SYN
Server  ->  Client    SYN-ACK
Client  ->  Server    ACK
```

Three packets, then the connection is up and SMTP starts.

### 2. SMTP Command Sequence

| Step | Command / Response | What it does |
|------|--------------------|--------------|
| 1 | 220 | Server ready, sends greeting |
| 2 | EHLO | Client identifies itself, asks for supported extensions |
| 3 | 250 (multi-line) | Server lists capabilities: SIZE, PIPELINING, etc. |
| 4 | MAIL FROM | Declares the sender address |
| 5 | RCPT TO | Declares the recipient address |
| 6 | DATA | Client signals it's about to send the message |
| 7 | 354 | Server says go ahead |
| 8 | message content | Headers (From, To, Subject, Date), then body, then a lone `.` on its own line to signal end |
| 9 | 250 | Server confirms message accepted |
| 10 | QUIT | Client closes the session |
| 11 | 221 | Server acknowledges, connection closes |

### 3. Message and Attachment

Once DATA starts, the client sends the full message. Headers come first, then the body, then the attachment encoded as Base64. Since there is no encryption, all of this is readable directly in the capture. To get the original attachment back, copy the Base64 block from the stream and decode it.

### 4. Session Close

```
Client  ->  Server    QUIT
Server  ->  Client    221 Bye
Client  ->  Server    FIN
Server  ->  Client    FIN-ACK
Client  ->  Server    ACK
```

Clean four-way TCP close after the SMTP session ends.

---

## What SMTP Is

SMTP (Simple Mail Transfer Protocol) is the standard protocol for sending email between servers and from clients to servers. It is a push protocol, meaning the sender pushes the message outward. Receiving email uses different protocols (IMAP or POP3).

It runs on port 25 for server-to-server relay and port 587 for client submission. The commands have not changed much since RFC 821 in 1982. The EHLO extension (RFC 5321) added capability negotiation, but the basic flow is the same.

---

## Security

This session has no encryption. The sender, recipient, subject, body, and attachment are all transmitted as plain text and fully readable in the capture. In a real environment you would use STARTTLS to upgrade the connection before any sensitive data is sent, or SMTPS on port 465 which wraps the entire session in TLS from the start.
