# OpenNGO-AI
# OpenNGO AI 🌱
### Open-Source Multi-Agent Platform for Predictive & Transparent Humanitarian Supply Chain Management

**Built by Gauransh Sharma** · Student Project · MIT License · Free for all NGOs forever
---

## What Is This?

OpenNGO AI is a free, open-source web platform that helps small and medium NGOs automate their operations using AI.

Any NGO in the world can take this code, connect it to their own database, and run their own version — for free, forever.

The platform has **3 AI agents** working together:

| Agent | What It Does |
|---|---|
| 💰 Finance Allocation Agent | Detects donations, auto-allocates funds, manages reserves, pays salaries |
| 📦 Procurement & Logistics Agent | Triggers purchase orders, tracks deliveries, ranks vendors by performance |
| 🔍 Transparency & Monitoring Agent | Logs every transaction publicly, generates weekly reports, fires risk alerts |

---

## What's In This Repo
openngo-ai/
├── openngo-ai.html        ← The entire app (one file — all 3 screens)
├── README.md              ← This file
└── screenshots/
├── public.png         ← Public transparency page
├── login.png          ← Employee login screen
└── dashboard.png      ← Full employee dashboard

The entire platform — public page, employee login, and full dashboard — lives in **one single HTML file**. No framework, no build step, no dependencies. Open it in a browser and it works.

---

## The 3 Screens

### 🌍 Public Transparency Page
- Anyone can visit, no login required
- Shows: live donation ticker, impact stats, fund allocation breakdown, public audit trail, weekly AI-generated report, upcoming events
- Builds donor trust through radical transparency
- Has an "Employee Login" button in the top right

### 🔐 Employee Login
- Email + password + role selector (Director / Manager / Staff)
- Wrong credentials = access denied
- Demo credentials are shown on screen for testing (remove before going live)

### 📊 Employee Dashboard (4 tabs)
- **Overview** — all agents status, live activity feed, real-time risk alerts
- **Finance** — fund allocation bars, donation log, predictive inventory requirements, salary disbursements, smart reserve ring
- **Procurement** — active purchase orders with delivery tracking, vendor rankings, cost savings
- **Transparency** — immutable audit log, AI-written weekly report, risk monitoring

---

## How to Run It Right Now

No install needed. Just:

1. Download `openngo-ai.html`
2. Double-click it to open in any browser
3. You're on the public page
4. Click **Employee Login →** in the top right
5. Use any demo credential:
   - `director@ngo.org` / `admin123` / Director
   - `manager@ngo.org` / `ops2025` / Manager
   - `staff@ngo.org` / `staff2025` / Staff
6. You're in the full dashboard

---

## How to Deploy on GitHub Pages (Free Hosting)

1. Create a GitHub account at github.com
2. Click **New Repository** → name it `openngo-ai` → set to Public
3. Upload `openngo-ai.html`, `README.md`, and the `screenshots/` folder
4. Go to **Settings → Pages → Source → main branch → / (root)**
5. Save — your site is live at `https://yourusername.github.io/openngo-ai`

That's it. Free forever. No server needed.

---

## What Is Fake Right Now (And How to Make It Real)

This is a prototype. Everything below is currently hardcoded demo data. Here is exactly what you need to replace, and how to do it.

---

### 1. 🔐 Login & Authentication

**Currently:** Passwords are hardcoded in JavaScript — anyone who reads the source code can see them. Fine for a demo, not for real deployment.

**How to make it real:** Use **Supabase Auth** (free at supabase.com):

```javascript
import { createClient } from '@supabase/supabase-js'
const supabase = createClient('YOUR_SUPABASE_URL', 'YOUR_SUPABASE_ANON_KEY')

async function doLogin() {
  const email = document.getElementById('login-email').value
  const password = document.getElementById('login-pass').value
  const { data, error } = await supabase.auth.signInWithPassword({ email, password })
  if (error) {
    document.getElementById('login-error').style.display = 'block'
  } else {
    const { data: profile } = await supabase
      .from('profiles').select('role').eq('id', data.user.id).single()
    go('dashboard')
  }
}
```

Create a `profiles` table in Supabase with columns: `id`, `email`, `role` (director/manager/staff).

---

### 2. 💰 Live Donation Data

**Currently:** Donations are hardcoded (Rahul M. ₹5,000, Priya K. ₹12,500, etc.)

**How to make it real:** Connect to your payment gateway webhook. When a donation arrives, it saves to Supabase automatically.

```sql
create table donations (
  id uuid default gen_random_uuid() primary key,
  donor_name text,
  amount numeric,
  currency text default 'INR',
  created_at timestamp default now(),
  status text default 'pending',
  allocated boolean default false
);
```

```javascript
async function loadDonations() {
  const { data } = await supabase
    .from('donations').select('*')
    .order('created_at', { ascending: false }).limit(10)
  data.forEach(d => renderDonationRow(d))
}
```

**Payment gateways (India):**
- **Razorpay** → Dashboard → Webhooks → add your site URL → listens for `payment.captured`
- **PayU** → similar webhook setup

---

### 3. 📦 Inventory & Stock Levels

**Currently:** Rice 420kg, Saplings 52, Dal 140kg — all hardcoded.

**How to make it real:**

```sql
create table inventory (
  id uuid default gen_random_uuid() primary key,
  item_name text not null,
  current_quantity numeric,
  unit text,
  reorder_threshold numeric,
  last_updated timestamp default now()
);
```

```javascript
async function loadInventory() {
  const { data } = await supabase.from('inventory').select('*')
  data.forEach(item => {
    const pct = item.current_quantity / item.reorder_threshold
    const status = pct < 0.3 ? 'Critical' : pct < 0.6 ? 'Low' : 'OK'
    renderInventoryRow(item, status)
  })
}
```

Tip: A Google Form filled by field staff after each drive, connected to Supabase via Zapier, works perfectly for small NGOs.

---

### 4. 📊 Fund Allocation (Finance Agent)

**Currently:** The 38% food, 22% plants etc. are hardcoded.

**How to make it real:**

```sql
create table allocation_rules (
  category text primary key,
  target_percentage numeric
);

create table transactions (
  id uuid default gen_random_uuid() primary key,
  category text,
  amount numeric,
  description text,
  created_at timestamp default now(),
  donation_id uuid references donations(id)
);
```

```javascript
async function allocateDonation(donationId, amount) {
  const { data: rules } = await supabase.from('allocation_rules').select('*')
  for (const rule of rules) {
    const allocated = (rule.target_percentage / 100) * amount
    await supabase.from('transactions').insert({
      category: rule.category,
      amount: allocated,
      description: `Auto-allocated from donation ${donationId}`,
      donation_id: donationId
    })
  }
}
```

---

### 5. 🛒 Purchase Orders (Procurement Agent)

**Currently:** PO-0047, PO-0048 etc. are hardcoded with fake vendors and statuses.

**How to make it real:**

```sql
create table vendors (
  id uuid default gen_random_uuid() primary key,
  name text,
  category text,
  contact text,
  avg_price_per_unit numeric,
  on_time_rate numeric,
  score numeric
);

create table purchase_orders (
  id uuid default gen_random_uuid() primary key,
  po_number text unique,
  vendor_id uuid references vendors(id),
  item text,
  quantity numeric,
  unit text,
  unit_price numeric,
  total_amount numeric,
  status text,
  expected_delivery date,
  created_at timestamp default now()
);
```

```javascript
async function checkAndTriggerOrders() {
  const { data: lowStock } = await supabase
    .from('inventory').select('*')
    .lt('current_quantity', 'reorder_threshold * 0.3')

  for (const item of lowStock) {
    const { data: vendor } = await supabase
      .from('vendors').select('*')
      .eq('category', item.category)
      .order('score', { ascending: false })
      .limit(1).single()

    await supabase.from('purchase_orders').insert({
      po_number: `PO-${Date.now()}`,
      vendor_id: vendor.id,
      item: item.item_name,
      quantity: item.reorder_threshold * 2,
      status: 'ordered'
    })
  }
}
```

Run this daily via a Supabase Edge Function (free, serverless).

---

### 6. 🌤️ Weather Forecast

**Currently:** "Rain forecast Wed afternoon" is hardcoded text.

**How to make it real:** Use **Open-Meteo API** — completely free, no API key needed:

```javascript
async function getWeatherForecast(latitude, longitude) {
  const lat = latitude || 28.6139  // Replace with your city coordinates
  const lon = longitude || 77.2090

  const response = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=precipitation_sum,weathercode&timezone=Asia/Kolkata&forecast_days=7`
  )
  const data = await response.json()

  data.daily.time.forEach((date, i) => {
    const rain = data.daily.precipitation_sum[i]
    if (rain > 5) {
      showAlert(`🌧️ Rain forecast on ${date} (${rain}mm) — consider adjusting outdoor plans`)
    }
  })
}
```

---

### 7. 🔮 Predictive Requirements (AI Forecasting)

**Currently:** "Need 680kg rice in 7 days" is a hardcoded guess.

**Simple version** — average of last 30 days usage:

```javascript
async function predictRequirements(itemName, daysAhead = 7) {
  const thirtyDaysAgo = new Date()
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30)

  const { data } = await supabase
    .from('inventory_usage_log').select('quantity_used, date')
    .eq('item_name', itemName).gte('date', thirtyDaysAgo.toISOString())

  const totalUsed = data.reduce((sum, d) => sum + d.quantity_used, 0)
  const avgDailyUsage = totalUsed / 30
  const predictedNeed = avgDailyUsage * daysAhead

  const { data: events } = await supabase
    .from('events').select('*')
    .gte('date', new Date().toISOString())
    .lte('date', new Date(Date.now() + daysAhead * 86400000).toISOString())

  const eventMultiplier = events.length > 0 ? 1.3 : 1.0
  return Math.ceil(predictedNeed * eventMultiplier)
}
```

**Advanced version** — use Claude API to forecast from historical data + events + weather:

```javascript
async function aiDrivenForecast(historicalData, upcomingEvents, weatherForecast) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 500,
      messages: [{
        role: 'user',
        content: `Based on historical usage: ${JSON.stringify(historicalData)},
                  upcoming events: ${JSON.stringify(upcomingEvents)},
                  weather: ${JSON.stringify(weatherForecast)},
                  predict how much rice, dal, and saplings this NGO needs in the next 7 days.
                  Return JSON only: { rice_kg: number, dal_kg: number, saplings: number }`
      }]
    })
  })
  const data = await response.json()
  return JSON.parse(data.content[0].text)
}
```

---

### 8. 📋 Audit Trail (Real Immutable Log)

**Currently:** 6 hardcoded rows with fake hashes.

**How to make it real:**

```sql
create table audit_log (
  id uuid default gen_random_uuid() primary key,
  timestamp timestamp default now(),
  event_type text,
  amount numeric,
  description text,
  reference_id uuid,
  hash text
);
```

```javascript
async function addAuditEntry(eventType, amount, description, referenceId) {
  const { data: lastEntry } = await supabase
    .from('audit_log').select('hash')
    .order('timestamp', { ascending: false }).limit(1).single()

  const prevHash = lastEntry?.hash || '0000000000000000'
  const payload = `${prevHash}|${eventType}|${amount}|${description}|${Date.now()}`

  const hashBuffer = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(payload))
  const hash = Array.from(new Uint8Array(hashBuffer))
    .map(b => b.toString(16).padStart(2, '0')).join('').slice(0, 16)

  await supabase.from('audit_log').insert({
    event_type: eventType, amount, description, reference_id: referenceId, hash
  })
}
```

---

### 9. 📧 Automated Reports to Director

**Currently:** "Sent to Director" is just a label. Nothing actually sends.

**How to make it real:** Use **EmailJS** (free, no backend needed):

```javascript
// Add to HTML: <script src="https://cdn.jsdelivr.net/npm/@emailjs/browser@4/dist/email.min.js"></script>

async function sendWeeklyReport(directorEmail, reportData) {
  await emailjs.send('YOUR_SERVICE_ID', 'YOUR_TEMPLATE_ID', {
    to_email: directorEmail,
    week_dates: reportData.weekDates,
    total_donations: reportData.totalDonations,
    beneficiaries: reportData.beneficiaries,
    cost_savings: reportData.costSavings,
    risk_flags: reportData.riskFlags,
    audit_url: 'https://yourusername.github.io/openngo-ai'
  }, 'YOUR_PUBLIC_KEY')
}
```

Schedule this every Monday using a Supabase Edge Function with a cron trigger.

---

### 10. 🤖 AI Chat Assistant (Already Real!)

The 🤖 chat button already calls the real Claude API and works in the Claude.ai preview. For GitHub Pages:

- **Option A (Simple):** Remove the API fetch and keep only the 7 fallback responses. Fine for demos.
- **Option B (Proper):** Create a Supabase Edge Function as a proxy that holds your API key securely. Never put an API key directly in public HTML.

---

## Full Stack for Real Deployment

| Layer | Tool | Cost |
|---|---|---|
| Frontend | This HTML file | Free |
| Database | Supabase (Postgres) | Free up to 500MB |
| Auth | Supabase Auth | Free |
| Hosting | GitHub Pages or Vercel | Free |
| Payment webhook | Razorpay | Free to integrate |
| Weather API | Open-Meteo | Free forever |
| Email reports | EmailJS or Resend | Free tier |
| AI assistant | Claude API | ~$0.003 per message |
| Cron jobs | Supabase Edge Functions | Free |

**Estimated monthly cost for a small NGO: ₹0 to ₹500** depending on AI chat volume.

---

## How Another NGO Can Set This Up

1. Clone or download this repo
2. Open `openngo-ai.html` in a text editor
3. Search for `YOUR_` and replace all placeholders with your own keys
4. Change the demo credentials to your real staff emails
5. Update event names, vendor names, and inventory items to match your operations
6. Deploy to GitHub Pages
7. Connect your payment gateway webhook
8. Start logging real data

---

## Roadmap

- [ ] Mobile app (React Native wrapper)
- [ ] SMS alerts to staff via Twilio
- [ ] Donor portal with personal donation trail
- [ ] Multi-NGO support
- [ ] Offline mode for field workers
- [ ] WhatsApp bot for inventory updates
- [ ] QR code receipts for donors

---

## License

MIT License — use it, modify it, deploy it, give it to other NGOs. Free forever.

---

## Credits

Built with ❤️ by **Gauransh Sharma** as a student project.

If this helped your NGO, a ⭐ on GitHub would mean the world.

> *"Originally built as a student project. Free for all NGOs forever."*
