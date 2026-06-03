---
name: payment-sessions
description: Explains the payment sessions pattern used across all payment processes.  The pattern isn't obvious from the code and caused a real bug.
version: 1.0.0
---

  # Payment Session Pattern

  All PaymentSession subclasses are serialized via Newtonsoft.Json with
  TypeNameHandling.All (see PaymentSessionService.cs). This REQUIRES a
  parameterless constructor on every subclass — without one, Json.NET calls
  the only available constructor with null arguments causing a crash on
  deserialization.

  Pattern: store only primitive IDs in the session. Re-load EF entities
  from the DB inside HandleSuccessAsync / HandleErrorAsync.

  ## Files
  - `Source/Web2/Models/PaymentSessions/` — all session types
  - `Source/Web2/Services/PaymentSessionService.cs` — serialization
  - `Source/Web2/Pages/PaymentForm.cshtml.cs` — consumes session
  - `Source/Payments/PaymentSession.cs` — base class

  ## Current session types
  - EventDonationPaymentSession
  - EventSignUpFeePaymentSession
  - ShopPurchasePaymentSession
  - UrlPaymentSession

  ---
  events-fundraiser-flow.md — The wizard + payment interception point is non-obvious:

  # Events Fundraiser Sign-Up Flow

  ## JoinFundraiser wizard
  Step1SelectEvent → Step2EditTarget → Step3EditPage
  All in `Source/Web2/Pages/Fundraisers/JoinFundraiser/`

  ## Sign-up fee interception
  Step3EditPage.cshtml.cs OnPostAsync intercepts if:
    ef.Event.SignUpFee > 0 && !ef.SignUpFeePaid

  Creates EventSignUpFeePaymentSession → redirects to /PaymentForm.
  EventFundraiser is always written to DB regardless of payment outcome.

  ## SignUpFeePaid
  Computed [NotMapped] property on EventFundraiser — true if SignUpPayment != null.
  EventFundraiser.SignUpPayment is set by EventSignUpFeePaymentSession.HandleSuccessAsync.

  ## Success/fail pages
  Source/Web2/Pages/Events/SuccessfulSignUp.cshtml.cs
  Source/Web2/Pages/Events/FailedSignUp.cshtml.cs
  Route: /Events/{event_id}/fundraisers/{event_fundraiser_id}/successful-sign-up

  ## Piranha page ID threading
  FundraiserView page requires ?id={piranhaPageId} in URLs.
  EventListItem accepts optional Guid? piranhaPageId in both constructors
  so templates can pass PiranhaPage.Id through for the Sponsor Someone link.