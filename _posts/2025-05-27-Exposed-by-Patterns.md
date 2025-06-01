---
layout: post
title: "Exposed by Patterns"
date: 2025-05-27
---
# Introduction: Patterns as a Threat
When we think of denial-of-service (DoS) attacks, we often imagine overwhelming floods of network traffic or systematically exploiting buffer overflows. However, modern adversaries can achieve similar (or even stealthier) results by weaponizing the very algorithms that drive our applications. Pattern-based side-channel attacks, also known as algorithmic complexity attacks, exploit worst-case inputs to force systems into expensive execution paths, all without injecting a single line of malicious code. This class of exploit utilizes valid, well-formed data that is explicitly crafted to trigger exponential behavior, essentially shutting down an application.  
  
Broadly, an algorithmic complexity attack leverages the natural relationship between input structure and runtime costs. Instead of bypassing code checks or hijacking memory, an attacker simply submits inputs that maximize backtracking in regex engines, hash-collision floods in dictionaries, or deep recursion in parsers. By observing CPU spikes, thread stalls, or increased response times, the adversary not only disrupts service but also gains insight into the application's internal logic. In internet-facing APIs, search forms, and validation endpoints where untrusted data is processed, these resource patterns become a potent side channel, leaking the shape of an application's code through every queued request and stalled cycle.  
  
In this post, I will examine how complexity attacks exploit patterns to both deny service and leak hidden behavior. You will see a step-by-step demonstration of a ReDoS (Regular Expression Denial of Service), learn about the ethical and legal considerations surrounding such attacks, and discover proven techniques for mitigating these covert threats. By the end, you will see why even a single crafted string can become a magnifying glass into a system's inner workings and how to close those cracks before attackers exploit them.  
<br /><br />
# The Threat Model
A robust threat model can help us understand precisely *what* algorithmic complexity attacks are and *how* algorithmic complexity attacks succeed. Unlike other side-channel attacks, these attacks require deeper assumptions about parsing logic, input validation, and runtime data structures.  

- **Valid (but Adversarial) Inputs:** The core feature of a complexity attack is that it uses *legal* input: the data conforms to expected types, encoding, and formats. However, the data is intentionally structured to produce worst-case execution behavior. This can include regex inputs that trigger catastrophic backtracking, hash keys that generate high collisions, or, broadly, queries designed to trigger complex execution paths. Ultimately, These inputs pass all validation checks as they are not malformed, but they are adversarial because of how they behave internally, not because of how they look externally.
- **Predictable Performance Degradation:** At the heart of the attack is a predictable relationship between input shape and resource consumption. Regex engines backtrack on ambiguous patterns. Hash tables slow down when lots of collisions occur. These relationships can be mathematically modeled and deliberately exploited. The attacker crafts inputs that push the algorithm toward its worst-case performance boundary, all without triggering conventional alarms.
- **Observable Side Effects:** Even without direct access to the server or codebase, an adversary can infer success by measuring a few key variables. The attacker should measure latency, which is when slow responses signal expensive execution paths, resource usage, which is monitored via local tools or side-channel measurement, and availability that may be affected when other users begin to experience degraded performance or outages. Like other side channels, this leakage is passive and indirect. It does not require compromising secrets, but it reveals how the system struggles.
- **Application Scope:** These vulnerabilities commonly appear in web forms and user-facing input validation, API gateways and authentication endpoints, content management systems with search or filtering features, and load balancers that parse HTTP headers or paths. Ultimately, though, any system that processes user input algorithmically is potentially at risk.

<br /><br />
# Exploiting Patterns to Cause a ReDoS (A Demo)
To make the risk concrete, let's walk through a simplified but realistic example of a Regular Expression Denial of Service (ReDoS). This class of vulnerability arises when regex engines use backtracking and are fed ambiguous patterns.  
  
### 1. Understanding Regular Expressions (Regex)
A regular expression is a concise, symbolic syntax used to define search patterns in text. Developers use regexes for tasks like validating input (e.g., email addresses, usernames), extracting substrings, and enforcing formatting constraints. While immensely powerful, regexes can be computationally expensive, particularly when the engine supports backtracking, a method of testing multiple ways to match a pattern when ambiguities arise.  
  
### 2. The Vulnerable Function
Let us consider what a possible input checker might look like:
```
def is_valid_username(username):
    pattern = r"^(a+)+$"
    return re.match(pattern, username)
```
This example defines a username validation function using Python's ``re`` module, which implements a backtracking regex engine. Lets breakdown the pattern ``r"^(a+)+$"``:
- ``^`` indicates the start of a string
- ``(a+)`` searches for one or more ``a`` character (captured as a group)
- ``+`` indicates one or more repetitions of that group
- ``$`` the end of the string
  
In essence, this pattern matches strings that consist only of one or more repeated 'a's.  
    
### 3. The Attack Input
Now consider this user input:
```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaX
```
This string contains a long sequence of ``a``'s followed by a non-matching character ``X ``. Thus, when this input is evaluated by the regex, the engine tries to match the entire string with the pattern ``^(a+)+$``. Because of the nested quantifiers (recursive ``(a+)+``structure), the engine doesn't know how to partition the repeated ``a``'s between the inner and outer loops. The engine then tries every possible grouping of the ``a``'s to find a match that satisfies both quantifiers and the end-of-string anchor. When it finally reaches the ``X``, it realizes the string does not match— but only after trying an exponential number of groupings. This behavior is called catastrophic backtracking.
> NOTE: Backtracking regex engines operate by trying different combinations of matches when multiple options are available. In a well-formed regex, this process terminates quickly. But when ambiguous quantifiers are nested, the engine may explore a decision tree of match attempts, which grows exponentially with input size. Because backtracking explores permutations recursively, each additional character drastically increases the number of steps required to fail.  

### 4. The Consequence
Even with modestly sized inputs, this pattern can consume seconds—or minutes—of CPU time. On single-threaded web servers or serverless environments, that means request queues build up and block legitimate users, CPU usage spikes, which exhausts autoscaling budgets, and threads are starved by halting asynchronous workloads. Moreover, this kind of behavior is invisible in unit tests or casual usage. Only adversarial inputs trigger the worst-case scenario.   
  
Critically, this isn't just a denial-of-service issue: the variation in response time under different crafted inputs leaks structural information about how the regex engine behaves. That turns this into a side channel that reveals the layout of internal logic without needing code access.
<br /><br />
# Ethical and Legal Considerations
As with all side-channel research, complexity attacks raise challenging questions about intent, permission, and proportionality. They are subtle, often misunderstood, and sometimes miscategorized as mere performance bugs, but their impact can be severe.  
### Ethical Ambiguity
- **Is this a vulnerability or just a bad design?** Complexity issues are often overlooked in development. While not bugs in the traditional sense, these flaws are security risks, and exploiting them to cause denial of service can be deeply disruptive.
- **Is it ethical to demonstrate or exploit this issue?** No. Any form of ReDoS testing should be conducted in isolated environments or under explicit scope within a penetration testing agreement. Triggering these behaviors against live systems can cause real harm, even unintentionally.
- **Is disclosure justified?** Yes. Responsible disclosure of complexity vulnerabilities is not only ethical, it's essential. If a system permits unbounded execution time due to input structure, users are at risk.
  
### Legal Landscape
- **CFAA:** Under the Computer Fraud and Abuse Act, sending crafted inputs to intentionally degrade system performance may be considered unauthorized access or causing damage to a protected computer, especially if availability is affected. Courts have interpreted denial-of-service attacks as violations under certain conditions.
- **International Frameworks:** Several jurisdictions now recognize resource exhaustion attacks as cybercrime, even if they do not involve access violations or data theft.
<br /><br />
# Mitigations and Best Practices
Protecting against complexity attacks requires both engineering discipline and operational controls. Fortunately, several defensive strategies are available.
- **Use Safer Regex Constructs**  
Avoid nested quantifiers and ambiguous patterns; instead, use linear-time regex engines such as Google's Re2 or Rust's regex crate. These engines guarantee worst-case linear behavior and reject vulnerable expressions at compile time.  
- **Apply Input Pre-Validation**
Perform basic checks on input length and structure before feeding it into complex parsers or validators. This prevents trivial overloads. For example, the check ``if len(input) > 256`` could act as a guard that dramatically reduces the attack surface.
- **Limit Execution Time and Enforce Rate Limits**
Use timeouts and execution quotas to constrain expensive operations. Many programming languages allow regex matching to run in separate threads with enforced time limits. Also, it detects repeated slow requests from the same client or pattern. Combine this with application-layer rate limiting to restrict the impact of brute-force complexity attempts.
<br /><br />
# Closing Thoughts
Algorithmic complexity is often seen as an abstract concern, but in the hands of a determined adversary, it becomes a weapon. As we've seen, a single crafted string can trigger exponential CPU usage, slow down entire services, and leak the structure of your application's logic, all without breaking a single rule or writing a single exploit. Ultimately, the deeper lesson is that secure systems fail safely. Performance bottlenecks are not just annoyances; they are vulnerabilities when predictable and unbounded. Complexity attacks exploit the assumptions we make about "normal" input and stretch them to a breaking point. If you are a developer, engineer, or security analyst, take time to understand your tools' worst-case behaviors. If you are a defender, include complexity resilience in your threat modeling. And if you are a researcher, continue pushing the boundaries of what "safe input" really means.
<br /><br />
# Additional Materials 
[Regular expression Denial of Service - ReDoS](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)  
  
[Understanding ReDoS Attack](https://www.geeksforgeeks.org/understanding-redos-attack/)  
  
<iframe
  width="560" height="315"
  src="https://www.youtube-nocookie.com/embed/bERhItWHEWM"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>  
