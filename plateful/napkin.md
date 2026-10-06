I'm a solo founder and a parent of two. I want to build a family meal planner with AI
suggestions. Working title: Plateful.

THE PROBLEM
Every Sunday night the same question: what are we eating this week? In our house one kid
has a tree-nut allergy, my partner went vegetarian last year, the other kid eats about six
things, and weeknights we have 30 minutes max. So planning means juggling screenshots of
recipes, a notes app, a half-remembered fridge, and a grocery list that three people edit
in a group chat. We still end up with takeaway on Wednesday and half a bag of spinach in
the bin on Saturday. The apps that exist are either recipe sites stuffed with ads, or
meal-kit companies that want to sell you boxes, or "AI meal plan" toys that suggest a
walnut salad to a family with a nut allergy.

WHAT IT IS
A shared household space where one person sets up the family once — who eats with us, what
each person can't or won't eat, how much time we have on which nights, our weekly food
budget — and then each week the AI proposes a realistic 7-day dinner plan that respects all
of it. We swap anything we don't like with one tap, the plan uses up what's already in the
fridge, and it produces one combined grocery list, grouped by supermarket aisle, that the
whole household shares and ticks off live in the store. Leftovers roll into lunches. Over
time it learns what the family actually likes.

WHO USES IT
1. The household organiser (usually one parent). Creates the household, invites others,
   sets everyone's dietary rules, budget and schedule, approves the weekly plan.
2. Other adult members (partner, roommate, grandparent who cooks on Tuesdays). See the plan,
   suggest swaps, add to and tick off the grocery list, rate meals.
3. Kids. They matter because their allergies and dislikes drive the plan, and older kids
   want to vote on dinners — but young kids shouldn't need their own account. I'm not sure
   how kids should be represented (see open questions).
One household per account for v1. Everyone in a household sees the same plan and list.
I as the operator only need basic support access, nothing more.

THE EXPERIENCE
Sunday evening, a notification: "Next week's plan is ready." The organiser opens it on
their phone: seven dinners, each showing cook time, rough cost, and little badges
confirming it's nut-free and has a vegetarian option. Thursday says "uses the spinach and
feta you already have". They swap Monday's curry for something faster with one tap, and
the grocery list updates instantly. On Saturday their partner is in the supermarket with
the same list open, ticking things off; the teenager adds "more yoghurt" from the couch
and it appears on the list in real time. Wednesday at 5pm, a nudge: "Tonight: 20-minute
pasta — take the chicken out of the freezer." After dinner the kids tap a thumbs up or
down. The key moment: the first week nothing gets thrown away and nobody asks "what's for
dinner?".

BUSINESS
Free tier: manual planning and the shared grocery list. Paid tier (monthly or yearly
subscription per household): the AI weekly plan, pantry-aware suggestions, and learning
from ratings. No ads, ever, and no selling family data — parents are sensitive about that
and it's a selling point. Why now: AI can finally plan around real constraints, food
prices went up so wasting food hurts, and every family already shares a grocery list
somewhere awful. First users: parents from my kids' school and online parenting groups.

SCALE & ENVIRONMENT
First year: several thousand households, 2 to 6 people each. Mobile-first — people plan on
the sofa and shop with the phone in one hand — but the organiser likes a bigger screen for
setup, so a responsive web app is fine for v1. The shared grocery list must feel instant
and must work in a supermarket with bad signal. Start in the US and UK, so units (cups vs
grams), currency and supermarket aisle names must not be hard-coded.

WHAT IT MUST LIVE ALONGSIDE
- An AI model for planning and suggestions. I don't mind which.
- Recipes: I want a starter library plus the ability to save recipes from any website by
  pasting a link. I don't know the rules on copying other sites' recipes.
- Online grocery ordering (Instacart, Tesco etc.) would be amazing later. Not v1.
- Calendar: nice-to-have to show dinner on the family calendar, not v1.
Otherwise it stands alone.

ONE YEAR FROM NOW, IT WORKED IF
- Families say they throw away noticeably less food and spend less.
- The organiser spends under 10 minutes a week on planning.
- Most paying households were invited by another household.
- It has never once suggested a meal that breaks a family member's allergy.

HARD BOUNDARIES
- Team: just me, building with AI coding tools. I want paying households within about
  three months.
- Budget: infrastructure under roughly $100/month until there's revenue — which means AI
  costs per household have to stay small.
- Safety: allergies are a hard rule, not a preference. The AI must never be the last line
  of defence; the app has to check its suggestions against the household's allergy list.
- Privacy: children use it, so their data is minimal and never used for anything but the
  family's own plan. No ads, no data selling.
- Platform: responsive web app for v1. No native apps, no app stores.
- No technology commitments — build it with whatever the blueprint recommends.

THINGS I'M NOT SURE ABOUT — please record these as open questions rather than guess
- How should kids be represented — profiles managed by a parent, or their own login once
  they're old enough? What age?
- Should it track a detailed pantry inventory, or just "use up what I tell you I have"?
- Breakfast and lunch too, or dinners only for v1?
- Is saving recipes from other websites legally OK, and in what form?
- Nutrition information (calories, macros) — useful or a liability for families?
