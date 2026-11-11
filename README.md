# Payments

The common payments vocabulary -- taking money, hosted checkout, refunds, stored payment
methods, payouts and disputes -- distilled from the APIs that already exist. Part of
[Schema Commons](https://github.com/api-commons/schema-commons), published by the
[API Commons](https://apicommons.org).

Built from the providers tagged Payments, Payment Gateway, Payment Processing, Card
Payments, Online Payments, Merchant Services, Acquiring, Payment Orchestration, Checkout,
Recurring, Embedded or Mobile Payments in [APIs.io](https://apis.io) that publish their own
OpenAPI. Only harvested contracts count -- nothing API Evangelist generated. Every number
below is a count of providers.

## What the market already agrees on

<!-- commons:summary -->
| Concept | Expose it | Readable schema | Core properties (share) | Collection named |
| --- | ---: | ---: | --- | --- |
| payment | 289 | 271 | id 75%, status 69%, amount 62%, created_at 52% | payments 189, charges 46, payment 42, charge 31 |
| checkout_session | 85 | 77 | id 78%, status 65%, created_at 51% | checkout 29, sessions 27, checkouts 23, payment-links 16 |
| refund | 214 | 201 | id 75%, amount 73%, status 63%, created_at 51% | refund 122, refunds 110, credits 15, reversals 12 |
| payment_method | 103 | 99 | id 69%, type 50% | payment-methods 41, payment_methods 24, cards 20, paymentmethods 13 |
| customer | 170 | 162 | id 83%, name 68%, email 66%, created_at 57% | customers 137, customer 31, buyers 8, payers 8 |
| payout | 122 | 110 | id 66%, status 66%, amount 64%, created_at 50% | payouts 70, settlements 29, payout 18, settlement 11 |
| dispute | 77 | 71 | status 72%, created_at 68%, id 68%, reason 68% | disputes 48, chargebacks 20, dispute 14, chargeback 7 |
| mandate | 18 | 16 | id 81%, status 56%, created_at 50% | mandates 13, mandate 5, direct_debits 1 |

Cohort: 692 providers with harvested OpenAPI, 438 crosswalks. Of the providers exposing a payment: 72% create, 67% list, 65% fetch, 22% update, 10% delete. Generated 2026-11-11 by build.py.
<!-- /commons:summary -->

Shares are of the providers with a readable resource schema for that concept. The
extended tier and the long tail of every concept are in [profile.yml](profile.yml).

Across every money-moving concept the pattern repeats: `id`, `status` and `amount` are
core, and `currency` sits just below (31-44%). Part of that gap is modelling -- 25 providers
nest currency inside an amount object (`amount: {value, currency}`) rather than beside it --
and the rest is providers that leave currency implicit in the account. The crosswalk
records which side each provider is on.

## Artifacts

- [openapi.yml](openapi.yml) -- OpenAPI 3.1 of the shared operations. No one serves it; it
  is the shape to map to. Lints clean against the
  [naming ruleset](https://github.com/api-commons/schema-commons/blob/main/ruleset/schema-commons-naming.yaml).
- [schema/](schema) -- JSON Schema 2020-12 per concept; each property carries
  `x-commons-tier`, `x-commons-share`, `x-commons-providers` and the spellings seen.
- [crosswalk/](crosswalk) -- one per provider, common name -> their name, with the harvested
  spec and operations each mapping was read from. Stripe's payment is a `charge`; Square's
  hosted checkout is a `payment-link`; Adyen's is a `session`.
- [adopters.yml](adopters.yml) -- who exposes which concept, linked to their APIs.io page.
- [concepts.yml](concepts.yml) -- the only hand-written input: concepts, path segments,
  aliases, reviewed exclusions, merged duplicates and shared specifications.
- [census/](census) -- the cohort and the raw census the profile is built from.

## Decisions

These are calls, not measurements, recorded so they can be revisited as the method matures.

- **"Acquired" is not a cohort tag.** It marks acquired companies in the catalog, not card
  acquiring. The API, AI, LLM and MCP "gateway" tags are not payment gateways either.
- **A hosted checkout session lives here, not in commerce.** The commerce context found
  `/checkout` to be a payment session almost everywhere and handed it over as
  `checkout_session`, alongside payment links and hosted pages.
- **`/sessions` is reviewed provider by provider.** Eight are hosted payment sessions (Citi,
  FastSpring, Klarna, Komoju, Plug, Satispay, Versapay, Zuora); sixteen are login, 3-D
  Secure, KYC, school-term or event sessions and are excluded with the reason in
  [concepts.yml](concepts.yml).
- **A card is a payment method only when it is saved on a customer.** The first build
  dropped `/cards` entirely, because 104 providers matched only through it returned `cvv`,
  `card_number` and a status -- issued cards. Building card issuing showed that was half
  right: a card under a customer (`/customers/{id}/cards`: Stripe, Square, Omise, PAY.JP,
  Pin, Shift4) is a stored way to pay. The segment is now qualified --
  `cards@customers|...` here, `cards!customers|...` in
  [card issuing](https://github.com/api-commons/card-issuing) -- and payment methods went
  from 86 providers to 103. `/tokens` is still not used.
- **`/transactions` is not a payment.** In this cohort it is as often a bank-ledger line or a
  card-programme feed, and the path alone cannot tell them apart.
- **A standard counts once.** Twenty-five UK banks republish the UK Open Banking
  specification byte-for-byte. Each is an adopter with its own crosswalk, but the vocabulary
  is measured once, under `shared_specs:`, because it was written once. (Their payment
  initiation resources, `/domestic-payments`, are a different shape from a card payment and
  do not match here -- a candidate for a payment-initiation concept.)
- **One company counts once.** Duplicate catalog listings found by
  `scripts/duplicates.py` (Finix, Billie, Mesh, Method and others) are merged under
  `duplicates:`.
- **Mandates (direct debit) added 2026-11-11.** 16 readable providers agree on `id`, `status` and `created_at`; the scheme (SEPA, Bacs, ACH) and the mandate reference stay below core, at 31% or less. Added because it was the one shared payments resource with no concept.

## Rebuild

```bash
../schema-commons/scripts/run.sh .
```

## License

Artifacts CC BY-NC-SA 4.0 ([LICENSE](LICENSE)); code Apache 2.0 ([LICENSE-CODE](LICENSE-CODE)).
