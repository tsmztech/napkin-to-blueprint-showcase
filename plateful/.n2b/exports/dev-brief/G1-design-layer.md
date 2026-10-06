# Part G1 — Design Layer

This package ships **design-agnostic**: no design system is part of the blueprint. The implementing team owns visual design, honoring any design preferences recorded in the brief's Constraints (Part A); Part G2 section 9 still records the styling-system and component-layer architecture decisions.

## Stated Preferences (from the Brief's Constraints)

## Constraints
- **Team / timeline:** the founder is building solo with AI coding tools and wants paying households within about three months. *Rationale:* a one-person venture that needs early revenue.
- **Budget:** infrastructure stays under roughly $100/month until there's revenue, so AI costs per household must stay small. *Rationale:* pre-revenue bootstrapping.
- **Safety (allergies):** allergies are a hard rule, not a preference. The AI must never be the last line of defence: the app checks every suggestion against the household's allergy list before anyone sees it. Meals show a "checked against allergies" badge with a standard "always check labels" disclaimer. *Rationale:* a single unsafe suggestion could harm a child and destroy trust.
- **Dietary rule strength:** allergies and religious rules (e.g. halal) are hard filters. Vegetarian can be set per person, with a "vegetarian option" on a shared meal. Dislikes are soft and learned from ratings. *Rationale:* stated by the founder when confirming how the household's rules apply.
- **Privacy / children (regulatory):** children use the product, so their data is minimal, parent-controlled and never used for anything but the family's own plan. There are no ads and no data selling. There is no medical or diet advice. *Rationale:* parents' sensitivity, and this is a selling point.
- **Platform:** a responsive web app for v1, with no native apps and no app stores. *Rationale:* a solo founder shipping fast.
- **Technology:** no technology commitments. Build it with whatever the blueprint recommends.
- **Design preference (no design system supplied):** friendly, calm and food-photo-led, with big tap targets for one-handed use in a shop. *Rationale:* people plan on the sofa and shop with the phone in one hand.

