# IRCTC Problem Discovery — Part A
 
## Summary
- Total problems documented: 6 (3 given + 3 self-discovered)
- Platform explored: irctc.co.in (live, as of June 2026)
- Devices used: Desktop Chrome, Mobile Safari
 
---
 
## Problem 1: Tatkal Booking Crashes at 10:00 AM [Given]
 
**What is broken:**
The system fails to handle the massive surge of concurrent users exactly at 10:00 AM (AC classes) and 11:00 AM (Non-AC classes). The backend becomes unresponsive, resulting in endless loading spinners, "Service Unavailable" errors, or unexpected logouts without any queue position or feedback.

**Affected users:**
Daily commuters and last-minute travelers depending on Tatkal tickets. Approximately millions of users hit the site at these exact times daily.

**Frequency:**
Daily exactly between 10:00 AM - 10:15 AM and 11:00 AM - 11:15 AM.

**Current flow — step by step:**
1. User logs into IRCTC web portal at 9:55 AM.
2. User enters source, destination, and selects date.
3. User selects "Tatkal" from the quota dropdown.
4. User waits until 10:00:00 AM and clicks "Search Trains".
5. Train list loads, user clicks on AC class availability (e.g., 3A).
6. User clicks "Book Now".
7. System shows an endless loading spinner "Please Wait..." followed by an unexpected redirect to the login page or a blank screen.
8. User logs back in, goes through the flow, and finds the Tatkal quota is fully waitlisted or exhausted.

**Where exactly it breaks:**
Step 7: The transition from the search results page to the passenger details page. The server times out due to connection exhaustion, but the frontend lacks any queue mechanism or graceful degradation, resulting in a dropped session.

---

## Problem 2: Search Filters Do Not Work Reliably [Given]

**What is broken:**
Applying search filters (like Train Class, Available only, or Departure Time) to train search results does not reliably update the UI, or the filters reset randomly if the user navigates to the passenger page and hits "Back". 

**Affected users:**
Planners and regular passengers searching across multiple train options along busy routes (e.g., Delhi to Mumbai) where filtering is necessary to find a suitable seat.

**Frequency:**
High. Present in almost every search session involving multiple filters.

**Current flow — step by step:**
1. User searches for trains between two major cities (e.g., NDLS to BCT) for a future date.
2. The search results list displays 30+ trains.
3. User clicks the "Available" and "Sleeper (SL)" filters on the left sidebar to narrow down options.
4. The train list updates correctly.
5. User clicks "Check availability & Fare" on a train, then clicks "Book Now".
6. User changes their mind, deciding to check another train, and clicks the browser's Back button or the on-page "Back" button.
7. User returns to the search results page, but all previous filters are cleared, showing the full unsorted list again.

**Where exactly it breaks:**
Step 7: The filter state is not preserved in the browser's session storage or URL parameters, forcing users to re-apply their preferences repeatedly.

---

## Problem 3: Seat Selection Resets [Given]

**What is broken:**
When checking a preferred berth (e.g., Lower Berth) during the passenger details entry, the selection often resets or is silently ignored if the system auto-refreshes or if the user adds an extra passenger.

**Affected users:**
Senior citizens and users with physical disabilities who rely heavily on Lower Berth allocations.

**Frequency:**
Moderate to High. Occurs frequently on the mobile web experience when scrolling or adding multiple passengers.

**Current flow — step by step:**
1. User selects a train and initiates the booking flow.
2. User reaches the "Passenger Details" page.
3. User enters their name, age, gender.
4. User selects "Lower" from the Berth Preference dropdown.
5. User clicks to add a second passenger.
6. User scrolls down and selects the payment option, then clicks "Proceed".
7. On the Review/Confirmation page, the berth preference for the first passenger is shown as empty or "No Preference".

**Where exactly it breaks:**
Step 7: The frontend form state does not strictly bind the berth preference selection to the payload sent to the review page, especially when DOM elements re-render (like adding a new passenger row).

---

## Problem 4: CAPTCHA Image Fails to Load [Self-Discovered]

**How I found it:** 
Trying to log in and occasionally on the final confirmation screen before payment, the CAPTCHA image box appears broken.

**Screenshot or description:**
A standard broken image icon appears where the alphanumeric CAPTCHA should be. (See assets/screenshots/captcha_error.png or visual description of missing CAPTCHA frame).

**What is broken:**
The dynamic generation of the CAPTCHA image occasionally fails and returns a 500 or 404 error. Clicking the "Refresh CAPTCHA" button spins but fails to fetch a new image, trapping the user in the login hurdle.

**Affected users:**
Any user attempting to log in or finalize a booking.

**Frequency:**
Intermittent, but spikes during high traffic periods (morning hours).

**Current flow — step by step:**
1. User enters username and password on the login modal.
2. User waits for the CAPTCHA image to load to type the text.
3. The CAPTCHA image throws a network error and displays a broken image icon.
4. User clicks the small "Refresh" icon next to the CAPTCHA.
5. The refresh icon spins indefinitely or simply does nothing.
6. User cannot submit the login form because CAPTCHA validation is required.
7. User is forced to hard-refresh the entire webpage and start over.

**Where exactly it breaks:**
Step 3 & 5: The `/eticketing/captcha` endpoint fails to serve the image, and the client-side error handling for the refresh click does not recover the state or provide a fallback.

---

## Problem 5: Ambiguous "Vikalp" Opt-in UI [Self-Discovered]

**How I found it:**
Scrolling through the Passenger Details page, just above the payment options, there is an easily missable checkbox for "Vikalp". 

**Screenshot or description:**
A checkbox labeled "Opt for Vikalp" that lacks clear context on what it entails before the user proceeds. (Description: Passenger details section, unlabeled toggle without an informational tooltip).

**What is broken:**
The UI provides a prominent checkbox to opt into the Vikalp (Alternate accommodation) scheme without adequately explaining that the user could be shifted to a completely different train and timing. The user only discovers the alternatives on a subsequent pop-up or after booking.

**Affected users:**
Waitlisted passengers who are unfamiliar with IRCTC terminology and accidentally opt into an unexpected travel schedule.

**Frequency:**
High visibility on every waitlisted ticket flow.

**Current flow — step by step:**
1. User selects a waitlisted train and reaches Passenger Details.
2. User fills out passenger information.
3. User scrolls down to "Other Preferences".
4. User sees "Opt for Vikalp" and checks it, assuming it means a better chance of a seat on the *same* train.
5. User proceeds to payment and completes the booking.
6. User checks PNR status later and realizes they have been shifted to a train departing 12 hours later.

**Where exactly it breaks:**
Step 4: The UI fails to provide immediate, contextual friction or a tooltip explaining that Vikalp involves alternate *trains*, not just alternate *seats*, leading to uninformed consent.

---

## Problem 6: Ambiguous "Session Expired" Post-Payment [Self-Discovered]

**How I found it:**
Testing the return redirect from a payment gateway. 

**Screenshot or description:**
A full white page with red text stating "Session Expired. Please log in again." after the bank deducts the amount.

**What is broken:**
If the payment gateway takes too long to confirm the transaction, IRCTC's session timer expires. When the user is redirected back to IRCTC, they are shown a generic logged-out error without confirmation of whether their ticket was booked or money refunded.

**Affected users:**
Users with slow 3G/4G network connections, especially on mobile devices or those using slower net banking gateways.

**Frequency:**
Moderate. Common in rural areas or during network congestion.

**Current flow — step by step:**
1. User selects train, fills details, and proceeds to payment.
2. User selects Net Banking and is redirected to their bank.
3. User completes OTP verification.
4. The bank processes the payment and redirects back to `irctc.co.in`.
5. The redirect takes longer than expected due to network latency.
6. The user lands on IRCTC and sees a "Session Expired" toast or page.
7. User receives a bank SMS saying money is deducted, but has no ticket confirmation (SMS/Email).
8. User has to log back in and manually check 'My Transactions' to find out the fate of their money/ticket.

**Where exactly it breaks:**
Step 6: The system strictly kills the active UI session rather than holding the transaction state asynchronously and providing a "Payment Processing" waiting room on return.
