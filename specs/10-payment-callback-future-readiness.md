- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Payment callback future readiness
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 7. Primary Pack 1 User Flows

## 7.7 Gateway payment completion flow

1. Student opens Pay Registration Fee screen
2. Student clicks Proceed to Payment Gateway
3. Hosted payment page is opened
4. After successful payment in future implementation, gateway sends trusted callback/webhook
5. System validates callback
6. System resolves enrollment by `payment_reference`
7. If valid and not already processed, fee status updates to `PAID`
8. Student dashboard and enrollment status view reflect paid status


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Future Payment Integration Module (out of scope for Pack 1)

Responsibilities:
- create payment gateway session/order
- redirect to hosted payment
- receive webhook/callback
- validate callback authenticity
- update enrollment fee status
- log payment events


# 10. Pack 1 Low-Level Design (LLD)

## 10.7 Future payment callback-compatible fee update sequence

This flow is defined now so later payment integration has a stable contract.

1. Student completes payment on hosted payment page
2. Gateway sends server-to-server callback/webhook
3. System validates callback authenticity
4. System resolves enrollment by `payment_reference`
5. System checks whether fee is already marked `PAID`
6. If not already processed, fee status updates from `UNPAID` to `PAID`
7. Callback handling must be idempotent
8. Payment event metadata should be logged in future payment pack

**Explicit prohibitions:**
- Student UI must not directly update fee status
- Admin approval flow must not directly update fee status


# 13. API Draft (Implementation-Oriented)

## 13.8 Future payment callback placeholder endpoint

### `POST /api/v1/payments/callback`

Reserved for future payment integration.

**Request (placeholder)**

```json
{
  "payment_reference": "ENR-REF-001",
  "status": "SUCCESS",
  "gateway_event_id": "evt_123",
  "amount": 200.0,
  "currency": "INR"
}
```

**Pack 1 note**
- endpoint may exist but stay internal/disabled until payment integration pack
- if enabled in dev/testing, it should update fee status to `PAID`

**Future implementation requirements**
- callback must be authenticated and signature-verified
- callback amount/currency should match enrollment snapshot
- callback handling must be idempotent
- duplicate gateway events must not double-process payment


# 14. Validation Rules and Business Rules

## 14.7 Gateway payment integrity rules

- Only trusted payment callback flow can mark fee status as `PAID`
- Student-facing endpoints cannot mutate fee status
- Admin enrollment approval endpoints cannot mutate fee status
- Future callback amount must match `registration_fee_amount_snapshot`
- Future callback currency must match `registration_fee_currency_snapshot`
- Callback processing must be idempotent

