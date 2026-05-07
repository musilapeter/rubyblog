---
title: "Algorithms Are Not Optional: Why Your 'Working' Code Will Collapse at Scale"
date: 2026-05-07 22:00:00 +0300
categories: [Software Engineering, Algorithms]
tags: [algorithms, data-structures, fintech, performance, oop, system-design]
image: /assets/img/posts/algorithms.png
---

Writing code without understanding algorithms is like building a skyscraper on a foundation of wet cardboard. A software engineer or a developer must know why a hash map outperforms a list, why recursion depth matters, and why sorting before searching changes everything — otherwise, their "working" code will collapse under real scale.

We treat algorithms like a whiteboard hazing ritual. Something you cram for interviews, then promptly forget once you've landed the job. *"I'll just use a library,"* we say. *"The framework handles that."*

Then production hits. Your simple payment loop starts timing out at 10,000 transactions. Your authentication check becomes a bottleneck. Your "elegant" OOP design turns into a nested nightmare of O(n²) hidden inside what looked like a clean method call.

Algorithms aren't academic toys. They are the difference between code that runs and code that survives.

---

## The OOP Trap: When Polymorphism Hides Complexity

Object-oriented programming promises clarity. You model real-world entities: `User`, `Account`, `Transaction`. You inherit, you override, you encapsulate. Beautiful.

But OOP without algorithmic thinking is **dangerous** because it hides performance costs behind pretty interfaces.

Consider a typical fintech account class:

```python
class Account:
    def __init__(self, account_id, transactions):
        self.account_id = account_id
        self.transactions = transactions  # list of Transaction objects
    
    def get_transactions_above(self, threshold):
        # Looks innocent
        result = []
        for tx in self.transactions:
            if tx.amount > threshold:
                result.append(tx)
        return result
```

Reading this, you think: *fine. O(1) per check.* But if you call `get_transactions_above` inside a loop over accounts, and each account has thousands of transactions, you've built **O(n²)** without realizing it.

Now add inheritance:

```python
class PremiumAccount(Account):
    def get_transactions_above(self, threshold):
        # Premium accounts get sorted results? 
        # Wait, that's O(n log n) every call.
        result = super().get_transactions_above(threshold)
        result.sort(key=lambda tx: tx.amount, reverse=True)
        return result
```

The reading engineer asks: *"Where is this method called? How often? Could we pre-sort once instead of per query?"*

Without algorithms literacy, you approve the PR. With it, you ask for a data structure change — maybe maintain a balanced tree or a heap of top transactions, updating incrementally.

---

## Data Structures Are Algorithm Delivery Systems

You cannot separate "algorithms" from "data structures." Choose the wrong structure, and your elegant algorithm becomes useless. Choose the right one, and a complex problem becomes trivial.

Let's take a real fintech example. I once worked on a payment orchestration system (call it *FinTech X*). We had to match incoming webhook events to pending transactions. The naive approach: store pending transactions in a list, iterate every time an event arrived.

```python
pending_transactions = []  # list of dicts

def match_event(event_id):
    for tx in pending_transactions:
        if tx['event_id'] == event_id:
            return tx
    return None
```

This was fine at 100 pending transactions. At 100,000, with 50 events per second, our CPU screamed. The fix? A hash map. **O(n) to O(1).** But you only reach for a hash map if you *know* that lookup by key is fundamental. That's algorithmic intuition.

Here's the reading exercise I gave my team:

```python
# Before reading: what's the algorithmic complexity of this?
def find_duplicate_payments(payment_list):
    duplicates = []
    for i in range(len(payment_list)):
        for j in range(i+1, len(payment_list)):
            if payment_list[i]['reference'] == payment_list[j]['reference']:
                duplicates.append((i, j))
    return duplicates
```

Then I'd show them the hash map version:

```python
def find_duplicate_payments_fast(payment_list):
    seen = {}
    duplicates = []
    for idx, payment in enumerate(payment_list):
        ref = payment['reference']
        if ref in seen:
            duplicates.append((seen[ref], idx))
        else:
            seen[ref] = idx
    return duplicates
```

The `if ref in seen` is **O(1)** average. The nested loops are **O(n²)**. That's the difference between a batch job that finishes in seconds vs. one that never finishes.

---

## Complex Systems Are Just Coordinated Algorithms

When you scale to a distributed fintech system, every component is an algorithm dressed up with networking.

- **Authentication** is a matching problem plus a time-based one-time password algorithm (TOTP).
- **Payment routing** is a shortest-path or weighted-selection algorithm.
- **Idempotency** is a deterministic hashing + lookup algorithm.
- **Fraud detection** is a sliding window or counting Bloom filter.

Let me walk you through the authentication overhaul at *FinTech X*.

### The Authentication Mess

Our original auth flow was simple: check username/password, generate a JWT, return it. But we needed step-up authentication for large payments (> $10,000). The requirement: after the user logs in, if they attempt a large transfer, we re-authenticate with a time-based code.

The naive implementation: store the user's OTP secret in the database, generate a code, compare. That works. But we had **5 million active users**. Reading the secret from DB for every large payment meant 5–10ms per check. At peak, that's hundreds of database calls per second.

We needed an algorithm: TOTP verification without a database round-trip for the secret retrieval. We cached secrets in Redis with a time-to-live, using a consistent hashing ring to distribute load. But the real algorithmic insight was this: TOTP is based on HMAC-SHA1 and a time counter. You can pre-compute a look-ahead window of valid codes.

Here's the simplified code we ended up with:

```python
import hashlib
import hmac
import time
import struct

def verify_totp(secret, user_code, window=1):
    # Read this: O(window) time, constant memory
    for i in range(-window, window + 1):
        counter = int(time.time() // 30) + i
        h = hmac.new(secret, struct.pack('>Q', counter), hashlib.sha1).digest()
        offset = h[-1] & 0x0F
        code = (struct.unpack('>I', h[offset:offset+4])[0] & 0x7FFFFFFF) % 1000000
        if str(code).zfill(6) == user_code:
            return True
    return False
```

Reading this algorithm taught my team **why window size matters** (security vs. usability), **why counter drift happens**, and **why you never roll your own crypto**. But also: why caching the secret in Redis with a short TTL is safe — because the algorithm itself includes a time window.

---

## Payment Idempotency: Where Algorithms Save Money

*FinTech X* processed millions of dollars daily. A duplicate payment due to a retry would be catastrophic. Idempotency is an algorithmic problem: given an incoming request with a unique idempotency key, decide whether to process it or return the previous result.

The data structure choice: a hash map that never grows unbounded. We used a **Bloom filter** for existence checks (to avoid disk reads), backed by a time-partitioned database. A Bloom filter tells you *"definitely not seen"* or *"maybe seen"* with probabilistic certainty.

```python
from pybloom_live import BloomFilter

# 1 million keys, 0.1% false positive rate
idempotency_filter = BloomFilter(capacity=1000000, error_rate=0.001)

def is_duplicate(key):
    if key not in idempotency_filter:
        return False  # definitely new
    # fall back to database lookup for "maybe seen"
    return db.exists(key)
```

The algorithm trades a small false-positive rate (0.1%) for massive speed and memory savings. And because we read the Bloom filter paper and understood the math, we knew exactly when to rebuild the filter and how to handle resets.

---

## The AI Bridge: Using LLMs to Debug Algorithmic Thinking

You don't have to memorize every algorithm. But you have to know what questions to ask. This is where AI shines as a reading partner.

**Prompt to use when reviewing an algorithmic implementation:**

```text
I have this function that is O(n^2) but needs to handle 100,000 items.
Explain why it's slow, then propose three alternative algorithms with 
their time and space complexity. Suggest which data structure would replace
the current list to achieve O(n log n) or better.
[Paste your slow function here]
```

**Another powerful prompt:**

```text
Act as a fintech principal engineer. This authentication code 
uses a linear scan over user sessions. At 1 million users, it takes 
5 seconds. Show me a hash-based approach. Then critique my 
understanding of hash collisions and rehashing.
```

I've seen mid-level engineers transform their code reviews by first feeding their own implementation to an AI with: *"Find the algorithmic inefficiencies in this code before I ask my senior to review it."* The AI catches the O(n²) hidden in a nested loop, or the recursive function with no memoization that's blowing the stack.

---

## The Bottom Line: Algorithms Are Not Optional for Complex Systems

When you write a simple CRUD app, you can ignore algorithms. But the moment you add authentication (hashing), payments (idempotency), or security (rate limiting), you are using algorithms whether you know it or not. The only question is whether you're using the right one or the wrong one by accident.

At *FinTech X*, the team that understood algorithms could reason about:

- **Why their payment retry logic needed exponential backoff** (and not constant retries)
- **Why a binary search on sorted transaction timestamps** was faster than a linear scan
- **Why their rate-limiting algorithm** (token bucket vs. fixed window) mattered for fairness

Reading algorithmic code — from open-source rate limiters, from TOTP libraries, from distributed consensus implementations — trains your brain to see patterns. Use AI to explain why a particular sort is stable or unstable. Use it to visualize recursion. But always, always read the output with a skeptical eye.

---

## Final Reading Challenge

Look at this code and ask yourself: *what algorithm should replace it for scale?*

```python
def authorize(user_id, permission):
    for role in user_roles[user_id]:
        for perm in role_permissions[role]:
            if perm == permission:
                return True
    return False
```

If you said **"hash set lookup"** or **"bitmask permission flags,"** you've passed. If you didn't, go read one more algorithm tonight.

Because writing code without algorithms is gambling. And in fintech, gambling with other people's money is not a career move.
