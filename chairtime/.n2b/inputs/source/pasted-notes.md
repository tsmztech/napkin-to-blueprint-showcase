I'm a solo founder. I want to build a booking app for independent beauty and wellness
professionals — barbers, nail techs, lash and brow artists, massage therapists, tattoo
artists — the ones who rent a chair or work from a home studio and run their whole
business out of Instagram DMs. Working title: Chairtime.

THE PROBLEM
They lose real money to no-shows and last-minute cancellations, and they spend their
evenings going back and forth in DMs to agree on a time. What they do today: Instagram
DMs to negotiate a slot, a paper diary or Google Calendar to hold it, a "send me $20 on
Venmo to hold your spot" message, and manual reminder texts the night before. It falls
apart constantly: double bookings, deposits that never arrive, no record when a client
disputes a no-show charge, and the pro is doing admin at 11pm. The tools that exist are
either built for multi-staff salons (heavy setup, monthly fee for features a solo pro
never uses) or free calendar links that can't take a deposit.

WHAT IT IS
A mobile-first booking page for ONE pro. A client opens the pro's link from their
Instagram bio, picks a service, picks a time from the pro's real availability, pays a
deposit by card, and gets a confirmation plus automatic reminders. The pro gets a clean
view of their day, sees who has paid, and never has to chase anyone. A no-show means the
deposit is kept, automatically, under the pro's own cancellation policy that the client
agreed to when booking.

WHO USES IT — TWO ROLES, CONFIRMED
1. The Pro (the owner). Sets up services, prices, deposit amount or percentage, working
   hours, buffer time between clients, and their cancellation window. Sees every booking,
   every client, and all the money. Can mark no-shows, reschedule, cancel and refund
   within policy, and block off time. The Pro is the only person who ever sees the client
   list.
2. The Client. Books, pays the deposit, reschedules or cancels within the policy window,
   and sees only their own upcoming and past bookings with that pro. Clients should not
   need to create a password-style account just to book — but I'm not sure what the
   lightest workable identity is (see open questions).
Strictly single-operator for v1. A pro with staff, or a salon with several chairs, is
explicitly out of scope — that's a different product. No other roles. I as the platform
operator only need basic support access (see a pro's account to help them), nothing more.

THE EXPERIENCE
A client taps the link in the pro's Instagram bio. Within one screen they see the pro's
name, services with prices and how long each takes, and the deposit rule in plain words.
They pick "Full set — $65 — 90 min", pick Thursday 2:30pm from times that are genuinely
free, enter their name and phone, pay a $20 deposit with a card, and they're done in
under a minute. A confirmation text lands immediately. Two days before, a reminder with a
one-tap "I'll be there / I need to reschedule". The pro, between clients, glances at their
phone: today's list, each with a paid badge, a client note, and how much is still due in
person. Nobody negotiated anything. The key moment is the first Friday night the pro
realises they haven't answered a single "are you free Saturday?" DM all week.

BUSINESS
Flat monthly subscription per pro, cheap enough that a single saved no-show pays for the
month. Absolutely no per-booking cut — these pros resent that and it's how they pick
tools. Deposits go to the pro, not to me. Why now: a huge number of these pros went solo
in the last few years and live on Instagram; paying a card deposit to a small business
is now normal; and the existing tools are either salon-sized or free-and-dumb. The first
pros will come from my own network (I know several) and from them referring peers.

SCALE & ENVIRONMENT
First year: a few hundred pros. Each pro has roughly 100–500 clients and 20–40 bookings a
week. Mobile-first web for both sides — the pro checks it on their phone between clients,
the client books from inside the Instagram in-app browser, so it must work well there.
Start in the US; timezone and currency must not be hard-coded because the UK, Canada and
Australia are obvious next markets. Reliability matters more than features: the moment it
silently double-books or loses a deposit, the pro is gone and tells their friends.

WHAT IT MUST LIVE ALONGSIDE
- The pro's personal Google Calendar or Apple Calendar: busy times there should block
  availability, and bookings made here should appear there.
- Text-message reminders (SMS). WhatsApp would be nice later, not v1.
- Card payments through an established payment processor. I never want to see or store a
  card number.
- Instagram is only the place the link lives — no Instagram integration needed for v1.
Otherwise it stands alone.

ONE YEAR FROM NOW, IT WORKED IF
- Pros say "I haven't had an unpaid no-show since I switched".
- Pros stop taking bookings by DM entirely — the link is the only way to book them.
- Most new pros arrive because another pro told them about it.
- Nobody has ever had a double booking or a lost deposit.

HARD BOUNDARIES
- Team: just me, building with AI coding tools. I want the first paying pro within about
  three months.
- Budget: infrastructure under roughly $100/month until there's revenue.
- Payments: card data is never stored or handled by my code — the processor owns that.
- Messaging: clients must explicitly agree to receive texts when they book, and reminders
  must respect that consent (US texting rules).
- Personal data: a pro must be able to delete a client's record on request, and clients'
  data is never visible to any other pro or client.
- Platform: mobile-first web only for v1. No native apps, no app stores.
- No technology commitments — build it with whatever the blueprint recommends.

THINGS I'M NOT SURE ABOUT — please record these as open questions rather than guess
- Should the balance after the deposit be payable in the app, or does it stay in-person
  (cash / card reader at the chair)?
- Should a pro be able to offer a waitlist for slots that open up from cancellations?
- Where does tipping fit, if anywhere?
- What is the lightest identity for clients — phone number plus a magic link? Something
  else? I don't want a signup wall.
- Recurring/standing appointments ("every 3 weeks") — v1 or later?
