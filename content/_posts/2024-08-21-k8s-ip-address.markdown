---
title: "Get K8s Pod IP with Python 3"
date: 2024-08-21 01:42:00 +0200
excerpt: Learn how to get a Kubernetes pod's IP using Python 3 in a minimal environment without curl.
categories:
  - DevOps
  - Kubernetes
tags:
  - Kubernetes
  - Python
  - Networking
  - Load Testing
  - Locust
splash_image: /assets/images/2024-08-21-k8s-ip-address.jpg
---

Today was my last day before a well-deserved vacation, and I was wrapping up by working on a load-test scenario using [Locust](https://locust.io/). Locust is a great tool for performing load testing, and I was all set to run a test against our production environment. However, I hit a roadblock—literally. The requests I was sending were returning a `403 Forbidden` status, indicating that my requests were being blocked.

## The Problem

After digging into the logs, I discovered that the requests were being cut off by Perimeter X, a security solution that prevents bot automations. I reached out to our internal team that handles Perimeter X, and they assured me that all necessary IP addresses were added as exceptions. However, they still needed my current IP address to double-check.

This is where the real challenge began. I needed to get the IP address from the Kubernetes pod running my Locust tests. Unfortunately, the pod environment was minimal, with no `curl` or `wget` available to make a quick HTTP request.

## The Solution: Python to the Rescue

Since Locust is written in Python, I had access to Python within the pod. I decided to leverage this by writing a quick one-liner in Python to retrieve the IP address:

```python
import requests; print(requests.get('https://api.ipify.org?format=json').json())
```

This command does exactly what I needed:
- **`requests.get`** sends a GET request to `https://api.ipify.org`, a public API that returns the client’s public IP address.
- **`.json()`** converts the response into a Python dictionary, from which I could easily print the IP address.

## Why Python?

Python is often available in containerized environments like Kubernetes due to its versatility. Even when you don't have common utilities like `curl`, Python's `requests` library can save the day by allowing you to interact with web services directly.

This solution was not only quick but also effective, allowing me to provide the internal team with the IP address they needed.

## Conclusion

In DevOps, you often encounter unexpected challenges—like needing an IP address without the usual tools. Python 3 proved to be a powerful ally in this scenario, helping me retrieve the necessary information and get back on track with my load testing.

Today’s experience also reminded me how much I enjoy solving these kinds of problems, which is why I decided to break my 1.5-year hiatus and share this update on my blog. Whether you're battling a security layer like Perimeter X or simply need to pull some data in a restricted environment, Python has your back.

Now, it’s time to switch off and enjoy some vacation time. Happy coding, and see you after the break!
