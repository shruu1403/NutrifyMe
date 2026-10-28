# NutrifyMe: MVP Product Requirements Document

> Instructions for the AI coding agent: Build exactly what this document describes, in the order given in Section 17. Do not add features outside scope (Section 3). Ask before changing the stack. Use TypeScript everywhere. After each build phase, make sure `npm run build` passes before moving on.

---

## 0. Current Project State (read first)

- The project folder `nutrifyme` **already exists**: Next.js (App Router, TypeScript, Tailwind, ESLint) created with `create-next-app`, and shadcn/ui initialised (Radix UI, Nova preset). **Do NOT re-scaffold or recreate the project.**
- Already installed: `@supabase/supabase-js`, `@supabase/ssr`, `zod`, `framer-motion`, `gsap`, `recharts`, `@dnd-kit/core`, `@dnd-kit/sortable`, `@react-pdf/renderer`, `@google/genai`, and `vitest` (dev). Install anything else that is needed.
- Folders already created: `lib/ai`, `lib/nutrition`, `lib/supabase`, `lib/pdf`, `components`, `supabase/migrations`, `supabase/seed`, `docs`.
- A Supabase project and a Gemini API key exist. Keys are in `.env.local` (never commit it, never print keys). Keep `.env.local.example` updated.
- Development machine is **Windows (PowerShell)**: give any terminal commands in PowerShell syntax.
- Work only on the phase the user names (see Section 17). Run `npm run build` at the end of each phase.

---

## 1. Product Overview

**NutrifyMe** is a nutrition-only web app for people in India. The user enters their details once, and the app generates a personalised daily and weekly Indian meal plan with calorie and macro estimates. The user logs what they actually eat by taking a photo of the plate (or typing it). An AI then shows what mistakes they made and what to do next, using real foods the user already eats.

**One-line pitch:** "Your AI nutrition coach that learns what you eat and fixes what you miss."

**Context:** This is a solo 7th semester college project, "AI for personalised nutrition". The MVP is due in 5 days and must work end to end for a live demo to teachers.

### Principles
1. AI understands, generates and explains. **Plain code does all the math** (BMR, calories, totals).
2. All food suggestions must be **real Indian meals with portions in household units** (roti, katori, glass), never vague advice.
3. The app **never diagnoses**. It only gives food guidance and only adapts to deficiencies the user says were confirmed by their own tests.
4. Nutrition only. No sleep, water, exercise or weight-loss-gadget tracking.

---

## 2. Target Users

- Indian students, working professionals and homemakers who want to eat better.
- Dietary types: Vegetarian, Non-vegetarian, Eggetarian, Vegan, Jain.
- Regional cuisines: North Indian, South Indian, Bengali, Gujarati, Maharashtrian, Punjabi, Rajasthani, East/Odisha, Pan-Indian.

---

## 3. Scope

### In scope for MVP
1. Auth (email + Google) with Supabase
2. Onboarding wizard and profile
3. Calorie and macro target calculation
4. AI daily and weekly meal plan generation
5. Plan dashboard with manual edit (swap, move, add, remove, regenerate)
6. Photo (snap) food logging with AI plate recognition and a confirm/edit step
7. Text food logging with AI parsing
8. Daily mistake and improvement report
9. Nutrition-only AI chatbot
10. Confirmed-deficiency mode (user-declared only)
11. Download plan as PDF
12. Animated, vibrant landing page
13. Indian food database (about 150 foods) with nutrient lookup
14. Legal and safety disclaimers

### Out of scope (do NOT build)
Sleep, water, exercise or step tracking; predicting or diagnosing deficiencies or diseases; wearable integration; payments; social features; native mobile apps; barcode scanning; multi-language UI (English only for MVP); admin panel; recipe videos; grocery ordering.

---

## 3A. Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js (latest stable, App Router), TypeScript |
| Styling | Tailwind CSS, shadcn/ui, Framer Motion |
| Landing animation | GSAP + ScrollTrigger (2D parallax food street) |
| Backend | Next.js Route Handlers (`/app/api/...`) and Server Actions |
| Database / Auth / Storage | Supabase (Postgres, Auth, Storage, Row Level Security) |
| Validation | Zod |
| AI | Gemini API (vision + text), accessed only through a provider wrapper in `lib/ai/` so the model can be swapped with env vars |
| Charts | Recharts |
| Drag and drop | @dnd-kit |
| PDF export | @react-pdf/renderer (or jsPDF as fallback) |
| Deployment | Vercel (frontend + API) and Supabase cloud |

### Environment variables (`.env.local`)
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
GEMINI_API_KEY=
GEMINI_MODEL=            # a Gemini Flash model, configurable
NEXT_PUBLIC_APP_URL=
```

---

## 4. User Flow (End to End)

1. **Landing page.** The user sees the animated "Indian food street" scroll experience and a "Get Started" button.
2. **Sign up / Log in.** Email + password or Google.
3. **Onboarding wizard (4 steps).** The user enters their profile. A progress bar shows the step.
4. **Target screen.** The app shows daily calories, protein, carbs, fat and fibre, with a short explanation of how they were calculated. The user clicks "Generate my plan".
5. **Plan generation.** A loading state, then a 7-day plan appears on the dashboard.
6. **Dashboard.** Today's meals, a progress ring for calories and macros, and quick buttons: Snap, Type, Ask Coach.
7. **Edit the plan.** The user swaps a meal, drags a meal to another slot, changes meal timings, regenerates a day or adds a custom meal.
8. **Log food.** The user snaps a photo, the AI lists what's on the plate with portions, the user confirms or edits, and the log is saved.
9. **Daily report.** The app compares eaten vs. target and the plan, shows mistakes and gives fixes.
10. **Coach chat.** The user asks nutrition questions and gets personal, grounded answers.
11. **Download.** The user downloads the weekly plan as a PDF.
12. **Confirmed-deficiency mode.** In Settings, the user declares a test-confirmed deficiency and future plans and advice adapt.

---

## 5. Feature Requirements

### 5.1 Authentication
- Supabase Auth: email/password and Google OAuth.
- Protected routes: `/dashboard`, `/plan`, `/log`, `/coach`, `/report`, `/settings`, `/onboarding`.
- Unauthenticated users go to `/login`. New users with no profile go to `/onboarding`.
- **Acceptance:** a user can sign up, log out, log in and only see their own data.

### 5.2 Onboarding (4-step wizard)
**Step 1, Basics:** name, age (14 to 80), gender (male/female/other), height (cm), weight (kg).
**Step 2, Goal:** `lose_weight`, `gain_muscle`, `maintain`. Optional target weight.
**Step 3, Lifestyle (for calorie calculation only):** activity level (`sedentary`, `light`, `moderate`, `active`).
**Step 4, Food habits:**
- Diet type: `veg`, `non_veg`, `eggetarian`, `vegan`, `jain`
- Region: `north`, `south`, `bengali`, `gujarati`, `maharashtrian`, `punjabi`, `rajasthani`, `east_odisha`, `pan_indian`
- Meals per day: 3, 4 or 5 (slots: breakfast, mid-morning snack, lunch, evening snack, dinner)
- Allergies (multi-select + free text), disliked foods (free text tags), budget (`low`, `medium`, `high`)
- Optional: "I have confirmed test results" with the checkboxes from 5.10.

**Validation:** Zod on client and server. Show friendly inline errors.
**Safety rule:** users under 18 get a notice and a calorie floor (no weight-loss targets for under 18; set goal to `maintain`).
**Acceptance:** completing the wizard saves a row in `profiles` and routes to the target screen.

### 5.3 Target calculation (deterministic, no AI)
Implement in `lib/nutrition/targets.ts` as pure, unit-tested functions.

- **BMR (Mifflin-St Jeor):**
  - Male: `10*kg + 6.25*cm - 5*age + 5`
  - Female: `10*kg + 6.25*cm - 5*age - 161`
  - Other: average of the two
- **TDEE** = BMR x activity factor (sedentary 1.2, light 1.375, moderate 1.55, active 1.725)
- **Goal adjustment:** lose = TDEE x 0.85; gain = TDEE x 1.10; maintain = TDEE
- **Safety floor:** never below 1200 kcal (female/other) or 1500 kcal (male). Round to nearest 10.
- **Protein:** lose 1.6 g/kg, gain 1.8 g/kg, maintain 1.2 g/kg (cap at 2.0 g/kg)
- **Fat:** 25% of calories / 9
- **Carbs:** remaining calories / 4
- **Fibre target:** 14 g per 1000 kcal
- Save all targets in `profiles` and show a plain-language explanation on screen.

### 5.4 AI meal plan generation
**Endpoint:** `POST /api/plan/generate` with body `{ scope: "week" | "day", date?: string }`.

**Behaviour:**
1. Load profile, allergies, dislikes, region, diet type, confirmed deficiencies, and the food database list (names only, filtered by diet type).
2. Call the LLM with structured JSON output (see Section 10.1).
3. **Validate** the JSON with Zod. For every meal item, **look up nutrients in the `foods` table** by `food_id`; the app computes calories and macros itself from the DB (never trust LLM-stated numbers).
4. Check each day's total is within 8% of the calorie target and protein within 15%. If not, retry once with feedback; if still off, scale portions in code or flag it.
5. Save to `plans` and `plan_meals`.

**Plan rules (put in the prompt):**
- Only Indian foods the user's region and diet type would actually eat.
- Respect allergies and dislikes absolutely. Never include a food from a conflicting diet type.
- Portions in household units (1 roti, 1 katori dal, 1 glass chaas).
- Variety: no main dish repeated on more than 2 days in a week.
- Each meal must include a short "why this meal" note (one line).
- Include timing for each meal slot (editable defaults: breakfast 8:00, snack 11:00, lunch 13:30, snack 17:00, dinner 20:30).

**Acceptance:** a first plan appears within about 15 seconds, with all 7 days, every meal showing items, portions, kcal and P/C/F.

### 5.5 Plan dashboard and manual editing
**Page `/dashboard`:**
- Greeting, today's date, today's target vs. consumed (calorie ring plus 3 macro bars)
- Today's meals as cards (time, items, kcal, P/C/F, status: planned / eaten / skipped)
- Quick actions: "Snap my plate", "Type what I ate", "Ask coach"
- Last 7 days mini-chart of calories vs. target

**Page `/plan`:** Week view (7 columns or horizontally scrollable tabs on mobile) and Day view toggle.

**Manual editing (required):**
- **Drag and drop** meals between slots and days (dnd-kit)
- **Edit time** of any meal slot
- **Swap meal:** opens a sheet showing 3 AI alternatives (similar calories and macros, same diet type); the user picks one. Endpoint `POST /api/plan/swap`.
- **Regenerate day:** `POST /api/plan/generate` with `scope: "day"`.
- **Add custom meal / food:** search the `foods` DB, choose quantity in units, nutrients computed automatically.
- **Remove meal.**
- After any edit, the day's totals update instantly and show a warning if over/under target by more than 10%.
- **Acceptance:** all edits persist after refresh.

### 5.6 Snap food logging (photo, the key feature)
**Page `/log`** with tabs: **Snap** | **Type**.

**Snap flow:**
1. User opens the camera (mobile) or uploads an image (desktop). Use `<input type="file" accept="image/*" capture="environment">`.
2. Client compresses the image (max 1024px on the long side, JPEG) and uploads it to Supabase Storage (`meal-photos`, private bucket, path `{user_id}/{uuid}.jpg`).
3. `POST /api/log/analyze` sends the image to the vision model with the prompt in 10.2.
4. The model returns detected items: `{ name, food_id_candidate, quantity, unit, confidence }`.
5. The server matches each item to the `foods` table (exact match, then fuzzy name/alias match). Unmatched items are flagged "not in database" and the user can search for a replacement.
6. The server computes nutrients from the DB.
7. **Confirm screen:** photo on top, list of items below. Each item has editable quantity (stepper), unit, a "change food" search, a delete button and an "Add item" button. Items with low confidence (<0.6) are highlighted in amber with "Please check".
8. The user picks the meal slot (breakfast, lunch, etc.) and date/time (default now), then taps **Save**.
9. The log is saved and the dashboard totals update.

**Type flow:** the user types e.g. "2 parathas with curd and a cup of chai". `POST /api/log/parse-text` uses the LLM to return the same item structure, then the same confirm screen.

**Rules:**
- Always show a confirm step; never auto-save AI guesses.
- Show a "Not sure? Edit" hint, and display total calories prominently.
- Handle failures gracefully (no food detected, blurry photo, API error) with a retry and the text-logging fallback.
- **Acceptance:** a photo of a typical thali produces an editable list of items with household-unit portions and calculated nutrients in under 10 seconds.

### 5.7 Daily mistake and improvement report
**Page `/report`** (with a Today / This Week toggle). Endpoint `POST /api/report/generate`.

The server first computes **facts in code**: totals vs. targets, per-meal deviations, plan adherence (eaten vs. planned), and patterns (e.g. low protein at breakfast on 4 of 7 days, high refined carbs at dinner). It then sends these facts to the LLM, which writes the explanation.

**Report sections:**
1. **Score** (0-100, computed in code from calorie and macro closeness and plan adherence)
2. **What went well** (2-3 points)
3. **Mistakes** (each with the evidence number, e.g. "Protein 38 g vs. 70 g target")
4. **What to do next** (specific Indian foods with portions, e.g. "Add 1 katori sprouts chaat or 100 g paneer at lunch")
5. **Tomorrow's tweak:** a button "Apply to tomorrow's plan" that regenerates tomorrow with the correction.

The LLM must only use numbers provided to it. Cache the report in `daily_reports`.
**Acceptance:** the report cites real numbers from the user's logs and gives concrete food fixes.

### 5.8 AI nutrition chatbot
**Page `/coach`** (also a floating chat button on the dashboard). Endpoint `POST /api/chat` (streaming).

- **System prompt:** a nutrition-only assistant for India. It refuses topics outside food and nutrition (politely, one line, steering back). It does not diagnose, prescribe medicine or doses, or comment on diseases beyond general food guidance.
- **Context injected each call:** profile summary, targets, today's logs and totals, the last 7 days' summary, confirmed deficiencies, and the last 10 messages.
- **Grounding:** when the user asks about a food's nutrition, the server looks up the food in the `foods` table and passes those values; the model must use them instead of guessing.
- Persist messages in `chat_messages`. Add suggested starter questions ("What should I eat tonight to hit my protein?", "Is rice at night bad?").
- Add a persistent small disclaimer: "Food guidance only, not medical advice."
- **Acceptance:** an off-topic question (e.g. cricket scores, sleep tips) is declined, and a food question gets a personalised, number-based answer.

### 5.9 Download plan as PDF
- Button on `/plan`: "Download PDF" (week or day).
- The PDF includes: name, goal, daily targets, a 7-day table (meal, items with portions, kcal, P/C/F), daily totals and a disclaimer footer.
- Generated client-side with @react-pdf/renderer. Clean, branded and printable.
- **Acceptance:** the downloaded PDF reflects manual edits.

### 5.10 Confirmed-deficiency mode (user-declared only)
**Settings page section:** "I have lab-confirmed results" with a checklist: Vitamin B12 low, Vitamin D low, Iron/Anaemia, Calcium low, Folate low, Zinc low, Protein low, Other (free text).

- Stored in `profiles.confirmed_conditions` (JSON array).
- Copy on screen: "NutrifyMe never detects deficiencies. Select only what a doctor or lab test has confirmed. This changes food suggestions only, and is not treatment."
- Effect: the plan prompt, swap suggestions, report and chatbot get the list and prioritise relevant foods (e.g. iron: ragi, spinach, jaggery, with vitamin C pairing tips; B12 for vegetarians: curd, paneer, milk, fortified foods; vitamin D: eggs, fortified milk, mushrooms). The assistant always adds "follow your doctor's advice for supplements."
- The app must never suggest supplements, dosages or medicines.
- **Acceptance:** toggling a condition and regenerating changes the plan's food choices accordingly.

### 5.11 Landing page
Route `/` (public). Vibrant Indian street-food visual theme.
- **Hero:** big headline, tagline, Get Started and Login buttons.
- **Scroll experience:** a 2D "walk through the Indian food street" built with GSAP ScrollTrigger and layered parallax. A walking character (Lottie or sprite) stays centred while stall layers (chai tapri, dosa counter, thali stall, sweet shop) scroll past. Each stall pops a feature card: **Plan**, **Snap**, **Coach**, **Report**.
- Bottom sticky chips to jump to each stall; honour `prefers-reduced-motion` (fall back to simple stacked sections).
- Sections after the scroll: How it works (4 steps), a screenshot/demo area, FAQ, footer with disclaimer.
- Performance: lazy-load animations, compress images (WebP), target Lighthouse performance above 80 on mobile.

### 5.12 Settings and profile
- Edit profile, goals, diet, region, allergies, dislikes, meal slots and timings (recalculates targets and prompts to regenerate the plan).
- Confirmed-deficiency settings (5.10), delete my data button, logout.

---

## 6. Pages and Routes

| Route | Access | Purpose |
|---|---|---|
| `/` | Public | Landing |
| `/login`, `/signup` | Public | Auth |
| `/onboarding` | Auth | 4-step wizard |
| `/targets` | Auth | Show targets, Generate Plan |
| `/dashboard` | Auth | Today view |
| `/plan` | Auth | Week/day plan, edit, PDF |
| `/log` | Auth | Snap / Type logging |
| `/report` | Auth | Daily/weekly mistakes and fixes |
| `/coach` | Auth | Chatbot |
| `/settings` | Auth | Profile, deficiencies, data |

Mobile-first layout. Bottom tab bar on mobile (Home, Plan, Snap, Coach, Report) and a sidebar on desktop.

---

## 7. Database Schema (Supabase / Postgres)

```sql
-- Enable extensions
create extension if not exists "pgcrypto";

create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  name text not null,
  age int not null check (age between 14 and 80),
  gender text not null check (gender in ('male','female','other')),
  height_cm numeric not null,
  weight_kg numeric not null,
  goal text not null check (goal in ('lose_weight','gain_muscle','maintain')),
  activity_level text not null check (activity_level in ('sedentary','light','moderate','active')),
  diet_type text not null check (diet_type in ('veg','non_veg','eggetarian','vegan','jain')),
  region text not null,
  meals_per_day int not null default 4 check (meals_per_day between 3 and 5),
  allergies text[] default '{}',
  disliked_foods text[] default '{}',
  budget text default 'medium' check (budget in ('low','medium','high')),
  meal_times jsonb default '{"breakfast":"08:00","snack_am":"11:00","lunch":"13:30","snack_pm":"17:00","dinner":"20:30"}',
  confirmed_conditions text[] default '{}',
  target_calories int, target_protein_g int, target_carbs_g int,
  target_fat_g int, target_fibre_g int,
  onboarding_done boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table foods (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  aliases text[] default '{}',
  category text,                 -- cereal, dal, vegetable, dairy, snack, sweet, beverage...
  diet_tags text[] default '{}', -- e.g. {veg,jain,vegan}
  region_tags text[] default '{}',
  serving_unit text not null,    -- 'roti','katori','glass','piece','plate'
  serving_grams numeric,
  calories numeric not null,     -- per ONE serving_unit
  protein_g numeric not null,
  carbs_g numeric not null,
  fat_g numeric not null,
  fibre_g numeric default 0,
  source text default 'IFCT-2017 approx'
);
create index foods_name_idx on foods using gin (to_tsvector('simple', name));

create table plans (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  week_start date not null,
  created_at timestamptz default now()
);

create table plan_meals (
  id uuid primary key default gen_random_uuid(),
  plan_id uuid not null references plans(id) on delete cascade,
  user_id uuid not null references profiles(id) on delete cascade,
  plan_date date not null,
  slot text not null,            -- breakfast, snack_am, lunch, snack_pm, dinner
  meal_time time,
  position int default 0,
  note text,
  status text default 'planned' check (status in ('planned','eaten','skipped')),
  items jsonb not null           -- [{food_id, name, quantity, unit, calories, protein_g, carbs_g, fat_g, fibre_g}]
);
create index plan_meals_user_date on plan_meals(user_id, plan_date);

create table food_logs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  logged_at timestamptz not null default now(),
  log_date date not null,
  slot text not null,
  source text not null check (source in ('photo','text','manual')),
  photo_path text,
  items jsonb not null,          -- same item shape + confidence
  total_calories numeric, total_protein_g numeric,
  total_carbs_g numeric, total_fat_g numeric, total_fibre_g numeric
);
create index food_logs_user_date on food_logs(user_id, log_date);

create table daily_reports (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  report_date date not null,
  scope text not null check (scope in ('day','week')),
  score int,
  facts jsonb,
  content jsonb,
  created_at timestamptz default now(),
  unique (user_id, report_date, scope)
);

create table chat_messages (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  role text not null check (role in ('user','assistant')),
  content text not null,
  created_at timestamptz default now()
);
```

### Row Level Security (required)
Enable RLS on every table except `foods`. For each user-owned table add policies so that `select/insert/update/delete` is allowed only where `user_id = auth.uid()` (for `profiles`, `id = auth.uid()`). `foods` is readable by all authenticated users, writable only via service role. The `meal-photos` storage bucket is private, with policies restricting access to the path prefix `auth.uid()`.

### Seed data
Create `supabase/seed/foods.json` and a seed script with **about 150 common Indian foods** covering rotis and breads, rice dishes, dals, sabzis, South Indian items (idli, dosa, sambar, upma, poha), North Indian items (paratha, chole, rajma, paneer dishes), Bengali and Gujarati items (khichdi, dhokla, thepla), eggs and non-veg (egg bhurji, chicken curry, fish curry), dairy (curd, chaas, milk, paneer), snacks, fruits, sweets and beverages (chai, coffee). Values are per one `serving_unit`. Example rows (approximate; verify against IFCT 2017):

| name | unit | grams | kcal | P | C | F | fibre |
|---|---|---|---|---|---|---|---|
| Roti (wheat, medium) | roti | 40 | 120 | 3.5 | 22 | 2.0 | 3.0 |
| Plain rice (cooked) | katori | 150 | 195 | 4.0 | 43 | 0.4 | 0.6 |
| Dal tadka | katori | 150 | 150 | 7.0 | 20 | 4.5 | 4.0 |
| Curd (dahi) | katori | 100 | 60 | 3.5 | 4.5 | 3.0 | 0 |
| Paneer bhurji | katori | 100 | 260 | 14 | 5 | 20 | 1 |
| Idli | piece | 40 | 60 | 2.0 | 12 | 0.3 | 0.5 |
| Masala dosa | piece | 120 | 220 | 4.5 | 33 | 8 | 2 |
| Poha | katori | 150 | 190 | 3.5 | 33 | 5 | 2 |
| Boiled egg | piece | 50 | 75 | 6.3 | 0.6 | 5 | 0 |
| Chaas | glass | 200 | 40 | 2.0 | 3.0 | 1.5 | 0 |

Add a README note that these values are approximate and sourced from IFCT-style references.

---

## 8. API Specification

All routes require an authenticated Supabase session, validate input with Zod and return JSON `{ data, error }`. Rate-limit AI routes (for example 20 calls per user per hour) to protect the API quota.

| Method + Route | Purpose |
|---|---|
| `POST /api/profile` | Create/update profile; compute and store targets |
| `GET /api/foods/search?q=` | Search foods (name, aliases) |
| `POST /api/plan/generate` | Generate week/day plan |
| `POST /api/plan/swap` | Return 3 alternative meals for a slot |
| `PATCH /api/plan/meal/:id` | Edit items, time, slot, status |
| `DELETE /api/plan/meal/:id` | Remove a meal |
| `POST /api/log/analyze` | Photo to detected items with nutrients |
| `POST /api/log/parse-text` | Text to detected items with nutrients |
| `POST /api/log` | Save a confirmed log |
| `GET /api/log?date=` | Logs for a date or range |
| `DELETE /api/log/:id` | Delete a log |
| `POST /api/report/generate` | Build facts, then AI report |
| `POST /api/chat` | Streaming chatbot response |
| `GET /api/chat` | Chat history |
| `DELETE /api/account` | Delete user data |

---

## 9. AI Architecture

`lib/ai/provider.ts` exposes `generateJSON<T>(prompt, schema, image?)` and `streamText(messages, system)`. Every structured call:
1. Requests JSON-only output.
2. Parses and validates with Zod.
3. Retries once on invalid JSON with the error message appended.
4. Returns a friendly error if it still fails.

Never send the user's name or email to the model. Send only what is needed (age, goal, diet, etc.).

---

## 10. Prompts

### 10.1 Plan generation (system)
```
You are an Indian clinical-style nutrition planner creating food plans (not medical advice).
Create a {scope} meal plan using ONLY foods from the provided FOOD_LIST (use their exact ids).
User: diet={diet_type}, region={region}, goal={goal}, meals_per_day={n}, budget={budget}.
Daily targets: {kcal} kcal, {protein} g protein, {carbs} g carbs, {fat} g fat.
Allergies (never include): {allergies}. Disliked (avoid): {disliked}.
Confirmed conditions declared by user (prioritise helpful foods, never mention supplements): {conditions}.
Rules:
- Real Indian meals a household would cook. Portions in the food's serving unit (e.g. 2 roti, 1 katori dal).
- Aim each day within 5% of calories and 10% of protein.
- Do not repeat a main dish more than twice in the week. Vary breakfasts.
- Include a one-line "note" per meal explaining why it fits.
Return ONLY JSON matching:
{ "days":[{ "date":"YYYY-MM-DD", "meals":[{ "slot":"breakfast|snack_am|lunch|snack_pm|dinner",
  "time":"HH:MM", "note":"string", "items":[{ "food_id":"uuid","quantity":number }] }] }] }
```

### 10.2 Plate recognition (vision)
```
You are analysing a photo of an Indian meal. Identify every distinct food item visible.
For each item return: name (common Indian name), food_id_candidate (null if unsure),
quantity (number), unit (one of roti, katori, glass, piece, plate, tbsp, cup),
confidence (0 to 1). Estimate portions in household units using plate/bowl size as reference.
Do NOT estimate calories. If the image has no food, return an empty list.
Return ONLY JSON: { "items":[{ "name":"", "food_id_candidate":null, "quantity":0, "unit":"", "confidence":0 }],
 "notes":"short remark about uncertainty" }
```

### 10.3 Text parsing
Same output schema as 10.2; the input is the user's sentence. Normalise Hinglish names ("do roti", "ek katori dal").

### 10.4 Report writer
Input: JSON facts only (targets, totals, per-meal deviations, adherence, patterns, deficiencies). Instruction: "Use only the numbers provided. Write: went_well[], mistakes[{issue, evidence}], fixes[{food_suggestion with portion, reason}]. Use Indian foods the user eats. Do not diagnose or mention supplements. Return JSON."

### 10.5 Chatbot (system)
```
You are NutrifyMe Coach, a friendly nutrition assistant for Indian users.
Only discuss food, nutrition, meal planning and the user's food logs.
If asked anything else, reply in one short line that you only help with nutrition, and offer a food-related alternative.
Never diagnose, prescribe medicine or supplements, or give medical treatment advice; suggest consulting a doctor when appropriate.
Use the provided USER_CONTEXT and FOOD_FACTS; do not invent nutrition numbers.
Give specific Indian food suggestions with portions. Keep answers short and practical.
```

---

## 11. UI / UX Requirements

**Theme: Indian street-food festival.** Colourful, vibrant, friendly.
- **Palette:** cream `#FFF3DC` background; magenta `#C2185B`, mustard `#F5A623`, teal `#0F9D8A`, tomato red `#E53935`, deep navy `#1B1F3B` for text/dark surfaces.
- **Style:** chunky rounded buttons with dark outline and offset shadow, rounded cards, subtle pattern accents (rangoli/diamond), generous spacing.
- **Fonts:** a bold display font for headings (e.g. Baloo 2 or Rozha One) and a clean sans for body (e.g. Inter or Poppins) via `next/font`.
- **Components:** calorie ring, macro bars, meal cards with food emojis or illustrations, bottom-sheet modals, toast notifications, skeleton loaders.
- **Motion:** Framer Motion for page and card transitions, animated rings, confetti when daily targets are hit. Keep the dashboard fast; heavy animation only on the landing page.
- **Accessibility:** WCAG AA contrast, keyboard navigable, alt text, reduced-motion support.
- **Responsive:** design mobile-first (360px and up); the camera flow must work well on phones.
- **States:** every screen needs loading, empty and error states.

---

## 12. Safety, Privacy and Compliance

- Visible disclaimer in the footer, plan PDF, chatbot and settings: "NutrifyMe gives general food guidance, not medical advice."
- No diagnosis, no supplements or dosages, no extreme calorie targets (floors in 5.3), and no weight-loss targets for under 18.
- If a user enters a BMI under 18.5 with a weight-loss goal, block the goal and suggest `maintain` with a gentle note.
- Photos stored privately, deletable by the user; "Delete my data" removes profile, logs, photos and chats.
- Never expose service-role keys to the client. All DB access is checked by RLS.

---

## 13. Non-Functional Requirements

- Page load under 3s on 4G for dashboard; AI calls show progress UI.
- Plan generation under about 15s; photo analysis under about 10s.
- Image upload compressed under 500 KB before sending.
- TypeScript strict mode, ESLint clean, no secrets in git.
- Unit tests for `lib/nutrition/targets.ts` and nutrient total calculations.

---

## 14. Folder Structure

```
/app
  /(public)/page.tsx            landing
  /(auth)/login, signup
  /(app)/dashboard, plan, log, report, coach, settings, onboarding, targets
  /api/...                      route handlers from Section 8
/components   ui/, landing/, plan/, log/, chat/, charts/
/lib
  /ai          provider.ts, prompts.ts, schemas.ts
  /nutrition   targets.ts, totals.ts, match.ts (fuzzy food matching)
  /supabase    client.ts, server.ts
  /pdf         PlanPdf.tsx
/supabase      migrations/, seed/foods.json, seed.ts
```

---

## 15. Demo Data

Provide a "Load demo data" script: one demo user, a generated week plan, 5 days of logs (some with deliberate mistakes such as low protein breakfasts) and a prepared report. This guarantees the teacher demo works even if the network is slow.

---

## 16. Definition of Done (MVP)

- [ ] A new user can sign up, finish onboarding and see accurate targets
- [ ] A 7-day Indian plan is generated, and all totals come from the foods DB
- [ ] Meals can be swapped, dragged, re-timed, added and removed, and changes persist
- [ ] A plate photo returns an editable item list with calculated nutrients and can be saved
- [ ] Text logging works as a fallback
- [ ] The dashboard shows consumed vs. target for today
- [ ] The report lists real mistakes with numbers and food fixes
- [ ] The chatbot answers nutrition questions with personal context and refuses off-topic ones
- [ ] Confirmed-deficiency toggle changes plan and advice
- [ ] PDF download matches the edited plan
- [ ] Landing page has the scroll-driven food street animation
- [ ] RLS verified: user A cannot read user B's data
- [ ] Deployed on Vercel with the demo data loaded

---

## 17. Build Order (5 days)

**Day 1:** Project setup, Tailwind/shadcn, Supabase project, schema + RLS, auth, onboarding wizard, targets calculation with tests, foods seed.
**Day 2:** AI provider wrapper, plan generation + validation, `/plan` and `/dashboard` display.
**Day 3:** Snap logging (upload, vision analysis, DB matching, confirm screen), text logging, today's totals.
**Day 4:** Plan editing (swap, drag/drop, time edit), report generation, chatbot, deficiency mode.
**Day 5:** Landing page animation, PDF export, theme polish, demo data, deploy, test the full flow on a phone.

**Cut order if time runs short** (cut from the bottom first): landing animation (use a static hero) > drag and drop (use up/down buttons) > swap alternatives > weekly report. Never cut: onboarding, plan generation, snap logging, report, chatbot.

---

## 18. Post-MVP (final submission phase, for reference only)

RAG over IFCT/ICMR-NIN guidelines, weekly adaptive replanning, grocery list, 3D version of the landing page, accuracy evaluation of the snap feature on 50 to 100 real meal photos, and user testing with 5 to 10 people.
