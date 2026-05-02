# drip. — Claude Code Handoff Prompt
# Paste this entire file into Claude Code to continue building

---

## WHAT IS drip.

drip. is an AI fashion stylist app for Gen Z women in India and the UK.
Brand name: drip.
Domain: dripstyle.ai
Tagline: "Wear it before you buy it"

Core concept:
- User types a prompt ("office look, Delhi heat, want to look powerful")
- AI generates a personalised outfit suggestion based on their Body ID
- App shows shoppable product cards with Myntra (₹) and ASOS (£) prices
- User clicks buy → affiliate commission earned

NOT a wardrobe builder. NOT a community app. NOT a closet organiser.
JUST: prompt → look → buy. That's it.

---

## TARGET USER

Women aged 18-35 in:
- India (primary) — Myntra, Ajio, Fabindia, Libas affiliate links
- UK (secondary) — ASOS, Zara, M&S affiliate links

Pain point: Trying on 10 things and buying nothing. drip. solves this.
Differentiator vs ChatGPT: Visual output + shoppable links + Body ID personalisation

---

## WHAT'S ALREADY BUILT

### Supabase Database (LIVE — already deployed)
7 tables created and running:
1. users — auth, plan (free/plus/pro/unlimited/creator), country (IN/GB), currency
2. body_profiles — Body ID (e.g. DRIP-IN-00247), body shape, skin tone, undertone, shoulder width, waist definition, hip ratio, torso length, leg length, height, style prefs, budget, 4 photo URLs
3. usage — daily request tracking, daily limits by plan, monthly totals, resets midnight IST
4. outfit_history — every Claude conversation saved, prompt, response, products shown, tokens used, cost
5. affiliate_clicks — every product click tracked, brand, price, commission rate, converted boolean
6. subscriptions — Razorpay (India) + Stripe (UK) payment records
7. saved_looks — user's saved favourite outfits

RLS (Row Level Security) enabled on all tables.
Functions created: generate_body_id(), check_usage_limit(), increment_usage()

### Core library files (in /lib folder)
- lib/supabase.ts — Supabase client, all DB functions (getOrCreateUser, getBodyProfile, saveBodyProfile, checkUsageLimit, incrementUsage, saveOutfit, getOutfitHistory, toggleSavedLook, trackAffiliateClick)
- lib/claude.ts — Claude Haiku API integration, mock dev mode (IS_DEV check), getMockResponse(), generateOutfit(), parseOutfitResponse(), product matching by occasion keyword
- lib/store.ts — Zustand global state (user, bodyProfile, usage, currentResult, conversationHistory, outfitHistory)

### Design
The owner has already designed and hosted their own UI via Claude Code.
DO NOT suggest or change the visual design.
DO NOT redesign any screens.
ONLY add functionality, fix bugs, wire up logic.

---

## TECH STACK

Frontend: React Native with Expo (Expo Router for navigation)
Styling: Owner's own design — do not touch
Backend: Vercel serverless functions (free hobby tier)
Database: Supabase (free tier) — project name: dripstyleai
AI: Claude Haiku (claude-haiku-4-5-20251001) — cheapest model, good enough for styling
State: Zustand
Payments: Razorpay (India) + Stripe (UK)
WhatsApp: Twilio (planned)
Body scan: expo-camera + MediaPipe (planned)
Hosting: Vercel

---

## ENVIRONMENT VARIABLES

These are in .env — DO NOT commit to GitHub:
EXPO_PUBLIC_SUPABASE_URL=
EXPO_PUBLIC_SUPABASE_ANON_KEY=
EXPO_PUBLIC_CLAUDE_API_KEY=
EXPO_PUBLIC_APP_ENV=development

When EXPO_PUBLIC_APP_ENV=development → use mock responses, do NOT call Claude API
When EXPO_PUBLIC_APP_ENV=production → call real Claude API

---

## SCREEN NAVIGATION FLOW

splash (index.tsx)
  → if not logged in → auth.tsx
  → if logged in + no body profile → scan.tsx (optional, can skip)
  → if logged in → style.tsx (main screen)
      → result.tsx (outfit result + products)
          → upgrade.tsx (modal — if over limit)
          → back to style.tsx
  → profile.tsx (accessible from style.tsx header)
  → upgrade.tsx (modal — accessible from anywhere)

---

## SCREENS TO BUILD / FIX

### app/index.tsx — Splash
- Check auth state from Supabase
- If session exists → route to style
- If no session → route to auth
- Show drip. logo while checking

### app/auth.tsx — Login
- Email + OTP via Supabase Auth (magic link)
- After login → check if body profile exists
- If body profile exists → go to style
- If no body profile → go to scan (with option to skip)
- Keep it simple — email input + "Send login link" button

### app/scan.tsx — Body ID Scan
- 4 step camera flow: front, right, back, left
- Use expo-camera for live camera feed
- Show body outline SVG guide overlay on camera
- 3 second countdown before capture (show 3, 2, 1)
- After each capture show thumbnail of captured pose
- Progress bar showing steps completed
- After all 4 captured → "Analysing" screen with animation
- Then reveal Body ID card: DRIP-IN-XXXXX format
- Show all 8 metrics: body shape, skin tone, undertone, shoulders, waist, hip ratio, torso, legs
- For MVP: use Claude Vision to analyse the photos for skin tone and basic proportions, fill rest with reasonable defaults
- Save to body_profiles table using saveBodyProfile()
- Generate body_id using generate_body_id() Supabase function
- User can skip this entire flow

### app/style.tsx — Main Prompt Screen (CORE SCREEN)
- Text input for outfit prompt
- Quick vibe chips: Office, Date Night, Festive, Weekend, London Winter, Eid, Wedding Guest, First Day
- Usage bar showing requests remaining today
- If user has Body ID → show "Styled for DRIP-IN-XXXXX · Hourglass · Warm Medium" badge
- On submit:
  1. Check usage limit via checkUsageLimit()
  2. If over limit → show upgrade modal
  3. Call generateOutfit() from lib/claude.ts
  4. In dev mode → mock response (free)
  5. In prod mode → real Claude Haiku call
  6. Save to outfit_history via saveOutfit()
  7. Increment usage via incrementUsage()
  8. Navigate to result screen

### app/result.tsx — Outfit Result (KEY SCREEN)
- Parse outfit response using parseOutfitResponse()
- Show: outfit name, 3 bullet pieces, editor tip, sign-off
- Show model image (use Unsplash fashion photos, match to body type)
- Show 3 product cards with: image, brand, name, INR price, GBP price, affiliate tag, buy button
- Buy button → trackAffiliateClick() → open URL
- Like button → update outfit_history liked field
- Save button → toggleSavedLook()
- Share button → React Native share
- "Style me again" → back to style screen

### app/upgrade.tsx — Paywall Modal
Show 4 plans clearly:

FREE — ₹0/month
- 5 outfit suggestions/day
- Basic styling

PLUS — ₹49/month (£0.99)
- 25 suggestions/day
- Body ID scan
- Save looks

PRO — ₹99/month (£1.99)
- 75 suggestions/day
- Virtual try-on (coming soon)
- Weekly WhatsApp lookbook

UNLIMITED — ₹249/month (£3.99)
- No daily limits
- Everything in Pro
- Priority response

Highlight PLUS as recommended.
Razorpay for India, Stripe for UK.
For now → Razorpay/Stripe placeholder (show plan, log selection, no real payment yet)

### app/profile.tsx — User Profile
- Show Body ID card if scan complete
- Show plan + usage stats (requests today, this month)
- Show last 10 outfit history items
- Edit style preferences
- Country toggle (IN / GB) — changes currency display and product recommendations
- Sign out button → supabase.auth.signOut()

---

## USAGE LIMITS BY PLAN

free:      5 requests/day
plus:      25 requests/day
pro:       75 requests/day
unlimited: 9999 requests/day (effectively unlimited)
creator:   9999 requests/day

When limit hit → show upgrade modal with message:
"You've used all X free looks today ✦
Your drip. resets at midnight 🕛
Upgrade to Plus for 25 looks/day — less than one chai ☕"

---

## CLAUDE API DETAILS

Model: claude-haiku-4-5-20251001 (cheapest — use this always)
Cost: ~$0.002 per outfit suggestion
Mock mode: when IS_DEV=true, return mock response, cost = $0

System prompt structure:
1. Body ID metrics (from bodyProfile in Supabase)
2. Market context (India = Myntra/Ajio brands, UK = ASOS/Zara)
3. Output format: look name, 3 bullet pieces, editor tip, sign-off
4. Max 160 words

Always use prompt caching for system prompt to reduce costs by 70%.

---

## WHATSAPP BOT (PLANNED — build after app is stable)

Tech: Twilio WhatsApp API + Vercel serverless function
Endpoint: /api/whatsapp (POST webhook)
Flow:
1. User messages WhatsApp number
2. Twilio sends POST to /api/whatsapp
3. Check if user exists in Supabase by phone number
4. If new user → create user record
5. Check usage limit
6. If over limit → reply with upgrade message
7. Call Claude Haiku with their message
8. Reply with outfit + Myntra affiliate links
9. Save to outfit_history

The WhatsApp bot uses the SAME Claude integration as the app.
The SAME Body ID is used if they've scanned on the app.

---

## AFFILIATE REVENUE MODEL

Every product shown has an affiliate link.
When user taps buy → trackAffiliateClick() records the click.
Affiliate programmes to join:
- Myntra Partner Portal (5-15% commission)
- Amazon India Associates (4-10%)
- ASOS Affiliate Programme (5%)
- Ajio Affiliate

For MVP → use placeholder #links with real product images and prices.
Replace with real affiliate links when approved by programmes.

---

## REVENUE PROJECTIONS (context only — not to build)

Month 3 target: ₹35,000/month
Month 12 target: ₹3,30,000/month
Sources: affiliate (primary) + subscriptions (secondary) + brand deals (tertiary)

---

## WHAT TO BUILD RIGHT NOW

Priority order:

1. Fix any existing errors in the codebase
2. Wire auth flow (index → auth → style)
3. Wire style screen to Claude API (mock in dev, real in prod)
4. Wire result screen to show outfit + products
5. Wire usage tracking (check limit → increment → show bar)
6. Wire upgrade modal when limit hit
7. Body ID scan flow (basic version — 4 photos, save to Supabase)
8. Profile screen
9. WhatsApp bot (/api/whatsapp endpoint)

---

## IMPORTANT RULES

1. DO NOT change the visual design — owner has their own design
2. DO NOT use Expo Go for testing — use web (npx expo start --web)
3. ALWAYS use mock mode in development (EXPO_PUBLIC_APP_ENV=development)
4. NEVER commit .env to GitHub
5. ALWAYS use Claude Haiku — never Sonnet or Opus (too expensive)
6. India users → show ₹ prices, Myntra/Ajio links
7. UK users → show £ prices, ASOS/Zara links
8. All user data is private — RLS enforced in Supabase
9. Usage resets midnight IST for all users
10. Body ID format: DRIP-IN-XXXXX (India) or DRIP-GB-XXXXX (UK)

---

## GITHUB REPO

https://github.com/dripstyleai/DripStyleAi-App

---

## START HERE

Read the entire codebase first.
Then fix any TypeScript or import errors.
Then wire the auth flow.
Then get style.tsx → result.tsx working end to end in mock mode.
Test with: npx expo start --web

Ask me if anything is unclear. Let's build drip. 🚀
