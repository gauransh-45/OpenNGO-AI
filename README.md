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

## Is The AI Real?

**Honest answer — it is mixed.**

✅ **Actually real AI:** The 🤖 chat button in the bottom right calls the real Claude API. When you ask it a question, a real AI is answering.

❌ **Simulated in this prototype:** The 3 agents (Finance, Procurement, Transparency) are currently UI simulations. The numbers, alerts, and activity feed are hardcoded demo data. They look like AI agents but no logic is running behind them.

This is a **UI prototype** showing what a real AI-powered NGO platform would look like. The README below explains two things:
1. How to connect real data (database, donations, inventory, weather)
2. How to build the 3 agents as real autonomous AI that runs 24/7

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

1. Download `OpenNGO-AI_by Gauransh.Sharma.html`
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

## Part 1 — Connect Real Data

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

### 4. 📊 Fund Allocation (Finance Agent data)

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

---

### 5. 🛒 Purchase Orders (Procurement Agent data)

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

### 7. 📋 Audit Trail

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

### 8. 📧 Automated Reports to Director

**Currently:** "Sent to Director" is just a label. Nothing actually sends.

**How to make it real:** Use **EmailJS** (free, no backend needed):

```javascript
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

---

## Part 2 — Build the Real AI Agents

This is the big step. Right now the 3 agents are UI labels. Here is how you turn each one into a real autonomous AI that runs 24/7 without anyone touching it.

Each agent is built as a **Supabase Edge Function** — a free serverless function that runs on a schedule (like a cron job) and uses the **Claude API** to make real decisions.

---

### How Supabase Edge Functions Work
Your database (Supabase)
↓
Edge Function runs on a schedule (e.g. every 15 minutes)
↓
It reads your real data (donations, inventory, events)
↓
It calls Claude API with that data and asks what to do
↓
Claude thinks and responds with a decision
↓
The function writes the result back to your database
↓
Your dashboard shows the real result live

No human needed. It runs automatically, forever, for free.

---

### 💰 Real Finance Allocation Agent

**What it does when real:**
- Watches for new donations every 15 minutes
- Reads your allocation rules from the database
- Calls Claude API to decide the smartest split based on upcoming events, current stock levels, and reserve status
- Records every allocation to the transactions table
- Logs to audit trail
- Updates the dashboard automatically

**Supabase Edge Function code:**

```javascript
// supabase/functions/finance-agent/index.ts
// Deploy with: supabase functions deploy finance-agent
// Schedule with: supabase functions schedule finance-agent --cron "*/15 * * * *"

import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

const supabase = createClient(
  Deno.env.get('SUPABASE_URL'),
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')
)

Deno.serve(async () => {

  // Step 1 — Find unallocated donations
  const { data: newDonations } = await supabase
    .from('donations')
    .select('*')
    .eq('allocated', false)

  if (!newDonations || newDonations.length === 0) {
    return new Response('No new donations', { status: 200 })
  }

  // Step 2 — Get current context
  const { data: inventory } = await supabase.from('inventory').select('*')
  const { data: upcomingEvents } = await supabase
    .from('events').select('*').gte('date', new Date().toISOString())
  const { data: rules } = await supabase.from('allocation_rules').select('*')
  const { data: reserve } = await supabase
    .from('transactions').select('amount').eq('category', 'reserve')

  const totalReserve = reserve.reduce((sum, r) => sum + r.amount, 0)

  // Step 3 — Ask Claude how to allocate
  for (const donation of newDonations) {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': Deno.env.get('CLAUDE_API_KEY'),
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 500,
        messages: [{
          role: 'user',
          content: `You are a finance agent for an NGO. A donation of ₹${donation.amount} just arrived.
          
          Current inventory levels: ${JSON.stringify(inventory)}
          Upcoming events this week: ${JSON.stringify(upcomingEvents)}
          Default allocation rules: ${JSON.stringify(rules)}
          Current emergency reserve: ₹${totalReserve}
          Target reserve: ₹145000
          
          Decide the smartest allocation for this donation.
          If reserve is very low, allocate more to reserve.
          If a plant drive is coming up, allocate more to plants.
          If food stock is critically low, prioritise food.
          
          Return JSON only, no explanation:
          {
            "food": <amount in rupees>,
            "plants": <amount in rupees>,
            "salaries": <amount in rupees>,
            "reserve": <amount in rupees>,
            "operations": <amount in rupees>,
            "reasoning": "<one sentence>"
          }`
        }]
      })
    })

    const aiResponse = await response.json()
    const allocation = JSON.parse(aiResponse.content[0].text)

    // Step 4 — Write allocations to database
    for (const [category, amount] of Object.entries(allocation)) {
      if (category === 'reasoning') continue
      await supabase.from('transactions').insert({
        category,
        amount,
        description: `AI allocated from donation ${donation.id} — ${allocation.reasoning}`,
        donation_id: donation.id
      })
    }

    // Step 5 — Mark donation as allocated
    await supabase.from('donations')
      .update({ allocated: true }).eq('id', donation.id)

    // Step 6 — Write to audit log
    await addAuditEntry('allocation', donation.amount,
      `Donation ₹${donation.amount} allocated by Finance Agent — ${allocation.reasoning}`,
      donation.id)
  }

  return new Response('Finance Agent ran successfully', { status: 200 })
})
```

---

### 📦 Real Procurement & Logistics Agent

**What it does when real:**
- Runs every morning at 7am
- Checks all inventory levels against thresholds
- Looks at upcoming events to predict extra demand
- Calls Claude API to decide what to order, how much, and from which vendor
- Auto-creates purchase orders in the database
- Sends WhatsApp or email notification to the vendor
- Logs everything to audit trail

**Supabase Edge Function code:**

```javascript
// supabase/functions/procurement-agent/index.ts
// Schedule: runs every day at 7am
// supabase functions schedule procurement-agent --cron "0 7 * * *"

Deno.serve(async () => {

  // Step 1 — Check what is running low
  const { data: inventory } = await supabase.from('inventory').select('*')
  const lowStock = inventory.filter(
    item => item.current_quantity < item.reorder_threshold * 0.5
  )

  if (lowStock.length === 0) {
    return new Response('All stock levels OK', { status: 200 })
  }

  // Step 2 — Get upcoming events and vendors
  const { data: events } = await supabase
    .from('events').select('*').gte('date', new Date().toISOString())
    .lte('date', new Date(Date.now() + 7 * 86400000).toISOString())

  const { data: vendors } = await supabase.from('vendors').select('*')

  // Step 3 — Ask Claude what to order
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': Deno.env.get('CLAUDE_API_KEY'),
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 800,
      messages: [{
        role: 'user',
        content: `You are a procurement agent for an NGO.

        Low stock items: ${JSON.stringify(lowStock)}
        Upcoming events this week: ${JSON.stringify(events)}
        Available vendors: ${JSON.stringify(vendors)}

        For each low stock item:
        1. Decide how much to order (enough for 2 weeks + event buffer)
        2. Pick the best vendor (highest score, lowest price, good on-time rate)
        3. Calculate total cost

        Return JSON only:
        [
          {
            "item": "rice",
            "quantity": 300,
            "unit": "kg",
            "vendor_id": "<uuid>",
            "vendor_name": "AgriMart India",
            "unit_price": 58,
            "total_amount": 17400,
            "reason": "Stock at 30%, plant drive Sunday needs buffer"
          }
        ]`
      }]
    })
  })

  const aiResponse = await response.json()
  const orders = JSON.parse(aiResponse.content[0].text)

  // Step 4 — Create purchase orders in database
  for (const order of orders) {
    const poNumber = `PO-${Date.now()}-${Math.random().toString(36).slice(2,6).toUpperCase()}`

    await supabase.from('purchase_orders').insert({
      po_number: poNumber,
      vendor_id: order.vendor_id,
      item: order.item,
      quantity: order.quantity,
      unit: order.unit,
      unit_price: order.unit_price,
      total_amount: order.total_amount,
      status: 'ordered',
      expected_delivery: new Date(Date.now() + 2 * 86400000).toISOString()
    })

    // Log to audit trail
    await addAuditEntry('purchase_order', order.total_amount,
      `PO ${poNumber}: ${order.quantity}${order.unit} ${order.item} from ${order.vendor_name} — ${order.reason}`,
      null)
  }

  return new Response(`Procurement Agent created ${orders.length} orders`, { status: 200 })
})
```

---

### 🔍 Real Transparency & Monitoring Agent

**What it does when real:**
- Runs continuously every hour
- Scans all transactions for anomalies (unusual amounts, duplicate entries, budget overruns)
- Calls Claude API to analyse patterns and write the weekly report in plain English
- Fires risk alerts if stock, budget, or vendor performance drops below safe levels
- Sends the weekly report to Director and Operations Manager every Monday
- Everything is logged publicly to the audit trail automatically

**Supabase Edge Function code:**

```javascript
// supabase/functions/transparency-agent/index.ts
// Schedule: runs every hour
// supabase functions schedule transparency-agent --cron "0 * * * *"

Deno.serve(async () => {

  // Step 1 — Pull all recent data
  const oneWeekAgo = new Date(Date.now() - 7 * 86400000).toISOString()

  const { data: recentTransactions } = await supabase
    .from('transactions').select('*').gte('created_at', oneWeekAgo)

  const { data: inventory } = await supabase.from('inventory').select('*')
  const { data: openPOs } = await supabase
    .from('purchase_orders').select('*').neq('status', 'delivered')
  const { data: donations } = await supabase
    .from('donations').select('*').gte('created_at', oneWeekAgo)

  // Step 2 — Ask Claude to analyse for risks and anomalies
  const riskResponse = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': Deno.env.get('CLAUDE_API_KEY'),
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1000,
      messages: [{
        role: 'user',
        content: `You are a transparency and risk monitoring agent for an NGO.

        Recent transactions (7 days): ${JSON.stringify(recentTransactions)}
        Current inventory: ${JSON.stringify(inventory)}
        Open purchase orders: ${JSON.stringify(openPOs)}
        Recent donations: ${JSON.stringify(donations)}

        Analyse this data and:
        1. Identify any risk alerts (stock critically low, budget overrun, anomalous transactions, delayed POs)
        2. Check if any inventory item will run out before the next scheduled event
        3. Flag any transaction that looks unusual in amount or timing

        Return JSON only:
        {
          "risk_alerts": [
            {
              "severity": "critical|warning|info",
              "title": "Short title",
              "message": "Full description of the risk and recommended action"
            }
          ],
          "anomalies": [
            {
              "transaction_id": "<id>",
              "reason": "Why this looks unusual"
            }
          ],
          "overall_health": "good|warning|critical"
        }`
      }]
    })
  })

  const riskData = await riskResponse.json()
  const analysis = JSON.parse(riskData.content[0].text)

  // Step 3 — Save risk alerts to database
  for (const alert of analysis.risk_alerts) {
    await supabase.from('risk_alerts').insert({
      severity: alert.severity,
      title: alert.title,
      message: alert.message,
      resolved: false,
      created_at: new Date().toISOString()
    })
  }

  // Step 4 — Generate weekly report every Monday
  const today = new Date()
  if (today.getDay() === 1) {  // Monday
    const reportResponse = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': Deno.env.get('CLAUDE_API_KEY'),
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 1500,
        messages: [{
          role: 'user',
          content: `Write a clear, professional weekly report for an NGO director.

          Data for this week:
          - Donations: ${JSON.stringify(donations)}
          - Transactions: ${JSON.stringify(recentTransactions)}
          - Inventory status: ${JSON.stringify(inventory)}
          - Purchase orders: ${JSON.stringify(openPOs)}
          - Risk alerts this week: ${JSON.stringify(analysis.risk_alerts)}

          Write in plain English. Include:
          1. Executive summary (3 sentences)
          2. Total donations received and how they were allocated
          3. Procurement highlights and cost savings
          4. Risk flags and how they were resolved
          5. Recommendations for next week

          Be specific with rupee amounts. Be honest about any problems.`
        }]
      })
    })

    const reportData = await reportResponse.json()
    const reportText = reportData.content[0].text

    // Save report to database
    await supabase.from('weekly_reports').insert({
      week_start: oneWeekAgo,
      week_end: new Date().toISOString(),
      content: reportText,
      created_at: new Date().toISOString()
    })

    // Send email to Director (using EmailJS or Resend)
    await sendWeeklyReportEmail('director@yourngo.org', reportText)
  }

  return new Response(`Transparency Agent: ${analysis.risk_alerts.length} alerts, health: ${analysis.overall_health}`, { status: 200 })
})
```

---

### How to Deploy the Agents

Once you have Supabase set up:

```bash
# Install Supabase CLI
npm install -g supabase

# Login
supabase login

# Link to your project
supabase link --project-ref YOUR_PROJECT_REF

# Deploy all 3 agents
supabase functions deploy finance-agent
supabase functions deploy procurement-agent
supabase functions deploy transparency-agent

# Set your Claude API key as a secret
supabase secrets set CLAUDE_API_KEY=your_claude_api_key_here

# Schedule them
supabase functions schedule finance-agent --cron "*/15 * * * *"
supabase functions schedule procurement-agent --cron "0 7 * * *"
supabase functions schedule transparency-agent --cron "0 * * * *"
```

That's it. All 3 agents are now running 24/7 for free, making real AI decisions on real data.

---

## Full Stack for Real Deployment

| Layer | Tool | Cost |
|---|---|---|
| Frontend | This HTML file | Free |
| Database | Supabase (Postgres) | Free up to 500MB |
| Auth | Supabase Auth | Free |
| Hosting | GitHub Pages or Vercel | Free |
| AI Agents | Supabase Edge Functions | Free |
| Payment webhook | Razorpay | Free to integrate |
| Weather API | Open-Meteo | Free forever |
| Email reports | EmailJS or Resend | Free tier |
| AI decisions | Claude API | ~$0.003 per decision |

**Estimated monthly cost for a small NGO: ₹0 to ₹800** depending on how often the agents run and how many donations come in.

---

## How Another NGO Can Set This Up

1. Clone or download this repo
2. Open `openngo-ai.html` in a text editor
3. Search for `YOUR_` and replace all placeholders with your own keys
4. Change the demo credentials to your real staff emails
5. Update event names, vendor names, and inventory items to match your operations
6. Deploy to GitHub Pages
7. Set up Supabase and create all the tables listed above
8. Deploy the 3 Edge Functions and set your schedules
9. Connect your Razorpay webhook
10. Start logging real data — the agents take over from there

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
