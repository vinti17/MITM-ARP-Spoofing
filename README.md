# MITM / ARP Spoofing — TLS Interception Lab

A man-in-the-middle attack carried out in a controlled lab network for the Networks & Security course at Radboud University. The goal was to position a laptop between a victim machine and the gateway, intercept the victim's HTTPS login, and recover the credential it sent.

This was an authorized classroom exercise on an isolated lab network set up for the task. It is documented here as a portfolio write-up. The captured credential (a course "flag") has been redacted.

## Repository Contents

- `docs/report.pdf`: full attack report (methodology, commands, results)
- `screenshots/`: evidence from the interception
- `capture/mitm-capture.pcap`: packet capture taken during the attack

## What the Attack Shows

A client that does not properly validate TLS certificates can have its "encrypted" HTTPS traffic read by an on-path attacker. Here the victim submitted a login whose password was recovered in plaintext, because the client accepted a forged certificate.

## How It Works

1. **Get on-path (ARP spoofing).** ARP has no authentication, so forged ARP replies tell the victim that the attacker's MAC is the gateway, and tell the gateway that the attacker's MAC is the victim. Both now send their traffic to the attacker.

2. **Keep traffic flowing (IP forwarding).** The attacker forwards packets on to the real destination, so the victim's connection keeps working and the interception stays unnoticed.

3. **Redirect HTTPS (iptables).** Traffic destined for port 443 is redirected to a local port where the proxy listens.

4. **Break the encryption (transparent proxy).** The proxy presents its own forged TLS certificate. A secure client would reject it; the insecure client here accepted it, letting the proxy decrypt the traffic, read it, and re-encrypt it to the real server.

5. **Read the credential.** With TLS broken, the login POST was visible in plaintext, exposing the password field.

## Tools

`arpspoof`, `iptables`, `mitmproxy`, `tcpdump`, `arp-scan`

## The Lesson (Mitigation)

The attack only succeeded because the client trusted an untrusted certificate. Proper certificate validation would have caused the forged certificate to be rejected and the connection to fail, keeping the password encrypted. This is exactly why TLS certificate validation exists.

## Note

Carried out only against lab machines provided for this exercise, with authorization. The techniques are shown for educational purposes; running them against networks or systems you do not own or have permission to test is illegal.
