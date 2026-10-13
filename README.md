# NGINX Layer 7 DoS Mitigation & Resilience Validation

## Overview

This project documents how I **assessed, mitigated, and validated** the impact of a Slowloris-style application-layer Denial-of-Service attack against an NGINX web server in an authorized lab environment.

Rather than stopping at attack simulation, I focused on the full defensive cycle: establishing an unprotected baseline, monitoring TCP behavior, implementing targeted NGINX controls, rerunning the same attack, and validating whether the server responded more effectively.

The goal was simple: **understand the weakness, apply the right control, and prove that the control made a measurable difference.**

---

## Technologies Used

**NGINX • Ubuntu Linux • Netdata • Slowloris • TCP/IP • Linux Terminal**

---

## Attack Simulation

I launched a **500-socket Slowloris attack** against the local NGINX server to evaluate how the server handled a high number of intentionally slow, incomplete connections.

Slowloris is effective because it does not rely on overwhelming bandwidth. Instead, it attempts to exhaust server resources by keeping many HTTP connections open for as long as possible.

![500 Socket Slowloris Attack](./01-slowloris-500-socket-attack.png)

---

## Unprotected Server Analysis

Before implementing any additional mitigation controls, I used **Netdata** to monitor TCP connection behavior during the attack.

The unprotected test showed a sharp increase in concurrent connections, demonstrating how slow and incomplete requests could continue consuming server resources.

![Unprotected TCP Connections](./02-unprotected-tcp-connections.png)

This baseline was important because it gave me something concrete to compare against after mitigation.

---

## NGINX Mitigation Strategy

Based on the attack behavior, I implemented controls designed to reduce the server's exposure to connection exhaustion.

The configuration introduced:

- Per-client connection limiting
- Shorter HTTP header timeouts
- Shorter HTTP body timeouts

```nginx
limit_conn_zone $binary_remote_addr zone=addr:10m;

limit_conn addr 10;

client_body_timeout 5s;
client_header_timeout 5s;
```

Before applying the changes, I validated the configuration:

```bash
sudo nginx -t
```

After confirming the syntax was successful, I reloaded NGINX:

```bash
sudo systemctl reload nginx
```

![NGINX DoS Mitigation Configuration](./03-nginx-dos-mitigation-config.png)

---

## Protected Server Validation

I then reran the **same 500-socket Slowloris attack** under the protected configuration.

Using the same attack conditions allowed me to compare server behavior fairly.

The protected Netdata results showed connections being terminated more aggressively rather than remaining continuously open, demonstrating the effect of the new timeout and connection controls.

![Protected TCP Connections](./04-protected-tcp-connections.png)

---

## Video Evidence

### Netdata Attack Monitoring

▶️ **[Watch the Slowloris Netdata Monitoring Demo](./05-netdata-slowloris-mitigation-demo.mov)**

### Protected Server Behavior

▶️ **[Watch the Protected Slowloris Netdata Demo](./06-protected-slowloris-netdata-demo.mov)**

---

## Results

The unprotected server allowed a large number of Slowloris connections to remain active for longer periods.

After implementing the NGINX mitigation controls, the server responded more aggressively to slow and incomplete requests.

The most important outcome was not simply that the attack still produced connection attempts. The important difference was **how the server handled those connections once the controls were in place**.

This reinforced a key defensive-security principle:

> A security control is only valuable when its effectiveness can be tested and validated.

---

## Security Analysis

This project demonstrated how an application-layer DoS attack can create meaningful availability risk without relying on massive network bandwidth.

It also demonstrated why defensive configuration matters.

Connection limits and request timeouts provide practical safeguards by reducing the amount of time an attacker can consume server resources through incomplete requests.

From a security perspective, the process was:

**Establish the baseline → Observe the attack → Identify the weakness → Implement controls → Retest → Validate the outcome**

---

## Skills Demonstrated

- Application-Layer DoS Analysis
- Slowloris Attack Simulation
- NGINX Security Hardening
- TCP Connection Analysis
- Netdata Monitoring
- Linux Administration
- Security Control Implementation
- Security Control Validation
- Defensive Security Testing
- Technical Risk Analysis
- Security Documentation

---

## Key Takeaway

This project strengthened my ability to move beyond identifying a security issue and into **evaluating risk, implementing a mitigation, and validating whether the control actually works**.

That distinction matters.

Cybersecurity is not only about detecting attacks.

It is about making informed decisions, applying the right controls, measuring the result, and improving resilience.
