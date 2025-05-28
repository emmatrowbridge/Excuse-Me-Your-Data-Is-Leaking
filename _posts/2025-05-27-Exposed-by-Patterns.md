---
layout: post
title: "Exposed by Patterns"
date: 2025-05-27
---
# Introduction: Patterns as a Threat
When we think of denial-of-service attacks, we often imagine overwhelming traffic floods or systematically exploiting buffer overflows and memory corruption. However, modern adversaries can achieve similar (or even stealthier) results by weaponizing the very algorithms that drive our applications. Pattern-based side-channel attacks, also known as algorithmic complexity attacks, exploit worst-case inputs to force systems into expensive execution paths, all without injecting a single line of malicious code. This class of exploit utilizies valid, well-formed data that is crafted specifically to trigger exponential behavior and essentially shut down an application.  
Broadly, an algorithmic complexity attack leverages the natural relationship between input structure and runtime costs. Instead of bypassing code checks or hijacking memory, an attacker simply submits inputs that maximize backtracking in regex engines, hash-collision floods in dictionaries, or deep recursion in parsers. By observing CPU spikes, thread stalls, or increased response times, the adversary not only disrupts service but also gains insight into the application's internal logic. In internet-facing APIs, search forms, and validation endpoints where untrusted data is processed, these resource patterns become a potent side channel, leaking the shape of an application's code through every queued request and stalled cycle.  
In this post, I will examine how complexity attacks exploit patterns to both deny service and leak hidden behavior. You will see a step-by-step demonstration of a ReDoS (Regular Expression Denial of Service), learn about the ethical and legal considerations surrounding such attacks, and discover proven techniques for mitigating these covert threats. By the end, you will see why even a single crafted string can become a magnifying glass into your system’s inner workings, and how to close those cracks before attackers exploit them.

# The Threat Model

# Exploiting Patterns to cause a Denial of Service Attack (A Demo")
Let us consider a simple web form that checks input against a regular expression. In Python, a vulnerable example might look like this:
```
import re

def is_valid_username(username):
  pattern = r"^(a+)+$"
  return re.match(pattern, username)
```
What is wrong? Well, this regex appears simple, but it is ca catastrophically vulnerable to a ReDoS attack. If an attacker submits a string like ``aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaX` ... the engine enters catastrophic backtracking, trying every possible combination before eventually failing. The result is exponential slowdown—CPU spiking, system lagging, and service potentially unavailable to others.

# Ethical and Legal Considerations
- **Is this a vulnerability or just bad design?** Complexity issues are often overlooked in development. While not bugs in the traditional sense, these flaws are security risks, and exploiting them to cause denial of service can be deeply disruptive.
- **Is it ethical to demonstrate or exploit this issue?** Ethically demonstrating a ReDoS—especially through responsible disclosure—is valuable. But launching such attacks against live systems without permission clearly crosses a line.
- **Is ignorance of algorithmic complexity an excuse?** Developers may not always understand the complexity implications of the tools they use. But security-conscious design means understanding how bad things get under stress.
- Under the CFAA, complexity attacks that cause denial of service can be prosecuted, even though they use “valid” inputs. Courts have treated these as unauthorized access or disruption.
- Intent again matters. Penetration testers with consent operate under protection. Unsanctioned actors—even if they don’t breach access controls—may be found liable for causing loss or damage.
- ReDoS attacks have been used in real-world attacks against major web services, and some nations now treat complexity abuse as a cybercrime, especially when deployed at scale.

# Mitigations and Best Practices
- Use safe regex engines that detect or avoid catastrophic backtracking.
- Prefer bounded or linear-time parsing algorithms when handling untrusted input.
- Apply rate limiting and input validation before pattern matching.
- Monitor for unusually long execution times or repeated heavy requests.

# Closing Thoughts
Algorithmic complexity isn’t glamorous, but it’s catastrophic in the wrong hands. Exploits like ReDoS reveal how assumptions about “normal input” can become dangerous when adversaries control the data. Complexity is a security concern, not just a computer science curiosity.
As developers and researchers, our job is not just to make systems work—but to make them fail predictably and safely. When time complexity becomes a weapon, it’s not just a performance issue. It’s a security one.

# Further Reading
