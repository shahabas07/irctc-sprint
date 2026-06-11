# Impact vs Effort Matrix
 
## The Matrix
 
|                   | Low Effort         | High Effort        |
|-------------------|--------------------|--------------------|
| **High Impact**   | 2. Persistent Filter States<br>3. Strict Seat Binding<br>5. "Vikalp" Explicit Consent Modal | 1. Virtual Queueing for Tatkal Surge<br>6. Payment Webhook Polling |
| **Low Impact**    |                    | 4. Fallback Audio CAPTCHA & Resilience |
 
## How I Scored Each Dimension
 
### Impact Scoring (1–5)
I scored Impact based on:
- Number of users affected (daily Tatkal surges vs edge-case network drops).
- Whether the problem blocks fundamental revenue-generating flows (like payment or form submission).
- Severity of consequence for the user (accidental Vikalp shifting is highly severe).
 
### Effort Scoring (1–5)
I scored Effort based on:
- Number of system components touched (pure frontend changes vs backend queuing refactors).
- Whether new infrastructure (like WebSockets, Redis, or Webhooks) is required.
- Risk of breaking existing monolithic core logic.
 
---
 
## Placement Justifications
 
### Feature 1: Virtual Queueing for Tatkal Surge — High Impact, High Effort
This completely removes the daily 10 AM anxiety for millions of daily commuters. However, it requires a massive infrastructural overhaul, likely involving Redis clusters or AWS SQS, and new WebSocket/polling frontend architecture. This is a massive engineering undertaking but fundamental to the platform's survival.
 
### Feature 2: Persistent Filter States via URL — High Impact, Low Effort
Filters resetting forces users to manually re-filter thousands of times a minute system-wide. Tying state to URL query parameters is a strictly client-side routing fix that requires zero database or API changes. It is a textbook Quick Win.
 
### Feature 3: Strict Seat Selection Data Binding — High Impact, Low Effort
Affects a highly vulnerable segment (Senior Citizens) who expect a lower berth but lose their selection due to a bad UI re-render. Refactoring the form to use strict controlled components is pure frontend engineering debt cleanup with no API integration effort. 
 
### Feature 4: Fallback Audio CAPTCHA & Resilience — Low Impact, High Effort
While being trapped outside the login gate is frustrating, it mostly spikes under severe load. Attempting to build an exponential backoff wrapper and an entirely new audio CAPTCHA microservice requires backend effort for a component that really should just be replaced entirely with modern invisible CAPTCHAs.
 
### Feature 5: "Vikalp" Explicit Consent Modal — High Impact, Low Effort
Vikalp frustration causes significant customer anger and TDR filings. Converting an ambiguous checkbox into a simple `<Dialog>` modal that warns users requires minor frontend layout effort but massively protects the user experience and reduces support overhead.
 
### Feature 6: Payment Webhook Polling — High Impact, High Effort
"Money deducted but ticket not booked" is the #1 financial complaint on the platform. Fixing it requires re-architecting the synchronous session flow to async webhooks and handling temporary transaction tokens across payment gateways, which is highly complex and touches strict financial compliance APIs.
 
---
 
## Recommended Sprint Order
1. **Feature 2: Persistent Filter States** — Immediate frontend UX relief with zero risk to bookings.
2. **Feature 3: Strict Seat Selection Data Binding** — Prevents critical real-world inconvenience for seniors.
3. **Feature 5: "Vikalp" Explicit Consent Modal** — A pure UI fix that prevents massive post-booking anger.
4. **Feature 6: Payment Webhook Polling** — Start scoping this now; it fixes the worst financial pain point.
5. **Feature 1: Virtual Queueing** — Quarter-long project. Put the infrastructure team on scoping.
6. **Feature 4: Fallback CAPTCHA** — Deprioritize in favor of evaluating fully invisible CAPTCHA alternatives later.
