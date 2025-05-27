---
title: "Breaking the Algorithm"
date: 2025-05-27
---
# Introduction: Exploiting Complexity to Deny Service
Not all vulnerabilities are bugs. Sometimes, systems break because they function exactly as designed—but under assumptions that attackers know how to violate. Algorithmic complexity attacks take advantage of this: they exploit inputs that push systems into their worst-case performance, overwhelming resources and leading to denial of service.
Unlike buffer overflows or injection attacks, these exploits don’t involve injecting malicious code. Instead, they craft malicious data that weaponizes the system’s own logic. For example, some data structures degrade from milliseconds to minutes with just the right input. Some regex engines take exponential time to evaluate carefully chosen strings. And some APIs, when given enough cleverly structured data, grind to a halt.
This post walks through an example of a complexity attack using a vulnerable regular expression engine. We’ll demonstrate how this seemingly innocent pattern-matching tool can be manipulated into burning CPU time, and we’ll examine the ethical and legal implications of weaponizing complexity.

# The Threat Model

# Regular Expression Denial of Service (A Demo)
Let us consider a simple web form that checks input against a regular expression. In Python, a vulnerable example might look like this:
```
import re

def is_valid_username(username):
  pattern = r"^(a+)+$"
  return re.match(pattern, username)
```
What is wrong? Well, this regex appears simple, but it is ca catastrophically vulnerable to a ReDoS attack. If an attacker submits a string like ``aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaX` ... the engine enters catastrophic backtracking, trying every possible combination before eventually failing. The result is exponential slowdown—CPU spiking, system lagging, and service potentially unavailable to others.

# Ethical and Legal Considerations
## Ethics
- **Is this a vulnerability or just bad design?** Complexity issues are often overlooked in development. While not bugs in the traditional sense, these flaws are security risks, and exploiting them to cause denial of service can be deeply disruptive.
- **Is it ethical to demonstrate or exploit this issue?** Ethically demonstrating a ReDoS—especially through responsible disclosure—is valuable. But launching such attacks against live systems without permission clearly crosses a line.
- **Is ignorance of algorithmic complexity an excuse?** Developers may not always understand the complexity implications of the tools they use. But security-conscious design means understanding how bad things get under stress.
  
## Legal
- Under the CFAA, complexity attacks that cause denial of service can be prosecuted, even though they use “valid” inputs. Courts have treated these as unauthorized access or disruption.
- Intent again matters. Penetration testers with consent operate under protection. Unsanctioned actors—even if they don’t breach access controls—may be found liable for causing loss or damage.
- ReDoS attacks have been used in real-world attacks against major web services, and some nations now treat complexity abuse as a cybercrime, especially when deployed at scale.

# Mitigations and Best Practices
- Use safe regex engines that detect or avoid catastrophic backtracking.
- Prefer bounded or linear-time parsing algorithms when handling untrusted input.
- Apply rate limiting and input validation before pattern matching.
- Monitor for unusually long execution times or repeated heavy requests.

# Further Reading

# Closing Thoughts
Algorithmic complexity isn’t glamorous, but it’s catastrophic in the wrong hands. Exploits like ReDoS reveal how assumptions about “normal input” can become dangerous when adversaries control the data. Complexity is a security concern, not just a computer science curiosity.
As developers and researchers, our job is not just to make systems work—but to make them fail predictably and safely. When time complexity becomes a weapon, it’s not just a performance issue. It’s a security one.
