# Web cache poisoning

In this section, we'll talk about what web cache poisoning is and what behaviors can lead to web cache poisoning vulnerabilities. We'll also look at some ways of exploiting these vulnerabilities and suggest ways you can reduce your exposure to them.

## What is web cache poisoning?

Web cache poisoning is an advanced technique whereby an attacker exploits the behavior of a web server and cache so that a harmful HTTP response is served to other users.

Fundamentally, web cache poisoning involves two phases:

1. The attacker elicits a response from the back-end server that inadvertently contains some kind of dangerous payload.
2. They ensure that this response is cached and subsequently served to the intended victims.

A poisoned web cache can be a devastating means of distributing various attacks such as XSS, JavaScript injection, open redirection, and more.

> #### Labs
> If you're already familiar with the basic concepts behind web cache poisoning and just want to practice exploiting them on realistic, deliberately vulnerable targets, check out the following:
> - [View all web cache poisoning labs](https://portswigger.net/web-security/all-labs#web-cache-poisoning)

## Web cache poisoning research

This technique was first popularized by the 2018 research paper, **"Practical Web Cache Poisoning"**, and further developed in 2020 with **"Web Cache Entanglement: Novel Pathways to Poisoning"**.

> #### Research
> - [Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning)
> - [Web Cache Entanglement: Novel Pathways to Poisoning](https://portswigger.net/research/web-cache-entanglement)

## How does a web cache work?

To understand how web cache poisoning vulnerabilities arise, it's important to understand how web caches work.

Caching reduces latency and server load by storing responses to particular requests. When a matching request is received, the cache serves the response directly without contacting the server.

![Normal cache behavior](https://portswigger.net/web-security/images/caching.svg)

### Cache keys

Caches determine whether a request can be served from the cache by using a subset of request components called the **cache key** (usually includes the request line and `Host` header).

Components not included in the cache key are called **unkeyed**. The cache ignores these, which can have serious implications, as we’ll see later.

## What is the impact of a web cache poisoning attack?

The impact depends on two key factors:

- **What the attacker gets cached**  
  If a harmful payload is cached, the impact depends on how dangerous it is. Attacks can be chained with others to escalate effects.

- **Traffic on the affected page**  
  The poisoned response is served only to users accessing the page while it's poisoned. If it's a high-traffic page, like a homepage, it can affect thousands.

> Note: Cache duration doesn’t limit the impact—attacks can be scripted to re-poison the cache repeatedly.

## Constructing a web cache poisoning attack

Steps to carry out a web cache poisoning attack:

1. [Identify and evaluate unkeyed inputs](#identify-and-evaluate-unkeyed-inputs)
2. [Elicit a harmful response from the back-end server](#elicit-a-harmful-response-from-the-back-end-server)
3. [Get the response cached](#get-the-response-cached)

### Identify and evaluate unkeyed inputs

These are inputs (e.g., headers) not considered in the cache key. Injecting payloads via these inputs can poison the cache.

You can manually find them by:

- Adding random headers
- Comparing responses with and without them (e.g., with Burp Comparer)

#### Param Miner

To automate this:

- Use [Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) extension in Burp.
- Right-click a request → “Guess headers”
- Results are shown in Burp’s "Issues" pane or the extension's "Output" tab.

Example: Param Miner found an unkeyed header `X-Forwarded-Host` on a homepage:

![Param miner](https://portswigger.net/web-security/images/param-miner.png)

> **Caution:** Always use a cache buster (unique parameter) to avoid accidentally poisoning the cache for real users. Param Miner can automate this.

### Elicit a harmful response from the back-end server

After identifying unkeyed inputs, understand how the server uses them. If the server reflects the input or uses it unsafely, it can be exploited.

### Get the response cached

Getting your response cached is essential. This can be tricky and depends on:

- File extension
- Content type
- Route
- Status code
- Response headers

Experiment with various requests to understand the caching behavior.

![web cache poisoning](https://portswigger.net/web-security/images/cache-poisoning.svg)

## Exploiting web cache poisoning vulnerabilities

There are two general categories:

- Flaws in **cache design**
- Quirks in **specific implementations**

> #### Read more
> - [Exploiting cache design flaws](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws)
> - [Exploiting cache implementation flaws](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws)

## How to prevent web cache poisoning vulnerabilities

The ideal way: **disable caching altogether**—though not always realistic.

Alternative strategies:

- Restrict caching to **purely static responses**
- Review and configure CDN behavior (e.g., disable unnecessary headers)
- Evaluate all **third-party technologies** for security implications

Additional precautions:

- Rewrite requests instead of excluding components from the cache key
- Avoid fat `GET` requests
- Patch client-side vulnerabilities even if they seem unexploitable
