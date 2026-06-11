# AI Feature Specification: Predictive Waitlist & Smart Vikalp Suggestions
 
## Problem It Solves
This AI feature directly addresses **Problem 5 (Ambiguous "Vikalp" Opt-in UI)**. Instead of blindly throwing users into the Vikalp bucket where they might get an inconvenient train, it uses machine learning to score their current waitlist odds and intelligently recommend specific, highly-probable alternative trains before they opt-in.
 
## Proposed Feature — User Perspective
When a user sees a "Waitlisted" train, an AI badge appears saying "42% Chance of Confirmation". When they click the badge, a modal opens stating: "Your waitlist is unlikely to clear. However, the AI suggests entering the Vikalp scheme specifically for 'Train 82415' which departs 1 hour later and has a 98% chance of clearing." The user can then deliberately select the AI-recommended Vikalp train, removing the ambiguity and fear of the classic Vikalp checkbox.
 
## Model or API Choice
**Custom XGBoost or LightGBM Classifier**.
Why: Waitlist prediction is fundamentally a tabular data classification problem (predicting outcome 1 or 0 based on day of week, train number, current WL number). Large Language Models (like GPT-4) are terrible and overly expensive for deterministic statistical forecasting. A fast, in-house tree-based model trained on tabular historical data offers millisecond inference times at low cost.
 
## Training or Input Data
**Training Data:**
- 5 years of historical IRCTC PNR data.
- Specific features: `Train Number`, `Date of Journey`, `Month`, `Class (3A/SL)`, `Current WL Number`, `Festive Season Flag`, `Days to Departure`.
**Where it comes from:**
- IRCTC's internal massive historical manifest database.
 
## How Output Is Shown to the User
The UI introduces an `AI PNR Predictor` pill component next to the waitlisted number.
 
```text
[ WL 45 ]  <-- [ AI: 42% Chance to Confirm ]
 
(If clicked):
+-------------------------------------------------+
| Intelligent Vikalp Alternatives                 |
| We recommend transferring to:                   |
| -> Train 12301 - Departs 18:00 (99% likelihood) |
| -> Train 12450 - Departs 21:00 (85% likelihood) |
| [ Accept Smart Vikalp ]   [ Reject ]            |
+-------------------------------------------------+
```
 
## Confidence Threshold and Fallback
- **Threshold:** If the model's confidence logic falls between 40% and 60% (a coin toss), it does not show the percentage. Instead, it shows "Uncertain".
- **Fallback:** If the ML API times out, the AI badge simply drops out of the DOM. The system gracefully degrades to the standard manual Vikalp checkbox from Feature Spec 5.
 
## Success Metrics
- Conversion rate of users explicitly adopting "Smart Vikalp" vs random Vikalp opt-ins.
- Model accuracy over time (how often did a 90% predicted PNR actually confirm).
- Reduction in post-reservation cancellations (since users trust the Vikalp allocation).
 
## Limitations and Risks
- **Black Swan Events:** Model will fail during unexpected political rallies or train cancellations.
- **Feedback Loops:** If the AI tells everyone to get on Train Y via Vikalp, Train Y will become full, inadvertently destroying the AI's own prediction. The model must have real-time access to the current dynamic inventory to penalize over-recommended trains.
