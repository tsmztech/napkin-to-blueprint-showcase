# Part G1 — Design Layer

This package ships **design-agnostic**: no design system is part of the blueprint. The implementing team owns visual design, honoring any design preferences recorded in the brief's Constraints (reproduced below and in Part A). Part G2 §9 still records the styling-system and component-layer architecture decisions.

## Stated Preferences (from the Brief's Constraints)

## Constraints
- **Team / timeline:** a solo founder building with AI coding tools, with a first paying freelancer within about three months. This bounds how much can be built and operated.
- **Budget:** infrastructure under roughly $100/month until there is revenue. Storage and bandwidth for big files have to be thought through.
- **Money handling (regulatory / trust):** payments go directly to the freelancer through the processor. The platform never holds or moves funds and never stores card data.
- **Record immutability (evidence):** accepted proposals, approvals and sent invoices must never be silently edited afterwards. They are the freelancer's evidence in scope disputes.
- **Privacy (regulatory):** a client can never see another client's anything, and freelancers must be able to export and delete their data. Personal data of worldwide clients falls under GDPR.
- **Platform:** web app for v1 (freelancer on desktop, clients on mobile browser). No native apps.
- **Technology:** no technology commitments. Build with whatever the blueprint recommends.
- **Design preference (brand):** clean and professional with lots of white space. Each freelancer sets their logo and brand colour, and possibly later a custom domain. No design system or artifacts were supplied.

