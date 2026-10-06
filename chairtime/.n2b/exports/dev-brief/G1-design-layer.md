# Part G1 — Design Layer

This package ships **design-agnostic**: no design system is part of the blueprint. The implementing team owns visual design, honoring any design preferences recorded in the brief's Constraints (Part A). Part G2 §9 still records the styling-system and component-layer architecture decisions.

## Stated Preferences (from the Brief's Constraints)

- **Team / timeline:** a solo founder building with AI coding tools, wanting the first paying pro within about three months. *Rationale:* one-person team and speed to first revenue.
- **Budget:** infrastructure under roughly $100/month until there's revenue. *Rationale:* pre-revenue solo venture.
- **Payments (regulatory / security):** card data is never stored or handled by the founder's code; the payment processor owns it. *Rationale:* "I never want to see or store a card number."
- **Messaging consent (regulatory):** clients must explicitly agree to receive texts when they book, and reminders must respect that consent. *Rationale:* US texting rules.
- **Personal data (regulatory / privacy):** a pro must be able to delete a client's record on request, and a client's data is never visible to any other pro or client. *Rationale:* client privacy and deletion-on-request obligations.
- **Platform:** mobile-first web only for v1; no native apps, no app stores. *Rationale:* where users are (phone, Instagram in-app browser) and speed to launch.
- **Scope:** strictly single-operator for v1; multi-staff pros and multi-chair salons are out of scope. *Rationale:* that's a different product.
- **Regulated-domain confirmation:** payments, US SMS consent and personal-data deletion were confirmed as the regulated areas. No health data: massage/tattoo intake forms are out of scope for v1.
- **Technology:** no technology commitments. *Rationale:* build it with whatever the blueprint recommends.


