You are asking the right questions. Let's clear the fog completely.

### 1. The "Lazy Reset" Explained (The Gym Analogy)

You are confused because you are thinking about **Time** (Midnight).
Stop thinking about Time. Think about **Events** (When the user knocks on the door).

Imagine a Gym. You have a pass for "2000 visits per day."
The Gym Receptionist (your Worker) does **not** stay awake until midnight to wipe everyone's card clean. That is a waste of time.

Instead, the Receptionist waits for you to show up.

**Scenario:**

* **Dec 27:** You enter the gym 2050 times. Your card says: `Count: 2050`, `Date: Dec 27`.
* **Dec 28, 1:00 AM:** You are sleeping. The database stays as `Count: 2050`, `Date: Dec 27`. **Nothing happens.**
* **Dec 28, 10:00 AM:** You show up and try to enter.

**The Receptionist (Worker) Logic:**

1. Receptionist looks at your card.
2. *"Hey, the date on this card is Dec 27. But today is Dec 28!"*
3. **Calculation:** "Yesterday you did 2050. That is 50 extra." -> **Writes '50' to the Bill Book (UsageHistory table).**
4. **Reset:** "Okay, I am wiping your card now." -> **Sets Card to `Count: 1`, `Date: Dec 28`.**
5. **Entry:** "You may enter."

**Why this is brilliant:**
If a user disappears for 5 days, your server does **zero** work. You only do the math when they actually come back to use the API.

---

### 2. The Security & "Infinite Bill"

**Your Security Idea (Password to View Key):**
**This is excellent.** Hiding the key behind a password check on the profile page is a standard, professional security practice (AWS does this for root credentials). It stops 90% of "I left my laptop open" hacks.

**The Financial Risk (My Warning):**
I will accept your decision to have **no hard limit**, but you must understand the **financial** risk, not the technical one.

* If a hacker steals the key (e.g., from the user's insecure code repository) and runs up a $5,000 bill...
* The user will call their bank and say "I didn't do this."
* **Stripe will refund them forcefully.**
* **AWS will still charge YOU for the server time.**

**Verdict:** Your "Password View" is approved. Just be ready to handle a dispute if a user gets hacked.

---

### 3. The Full Process (Stripe Integration)

Yes, **Stripe** is the standard. It handles the credit cards, the receipts, and the trust.

Here is the **Crystal Clear Workflow** we will build.

#### Phase 1: The First Purchase (Today)

1. **User:** Goes to `/pricing`, selects "Pro Plan" ($44/month).
2. **Website:** Calls Stripe, creates a **Checkout Session**.
3. **User:** Enters credit card on Stripe.com (secure).
4. **Stripe:** Redirects user back to your site -> `/payment/success`.
5. **Behind the Scenes (Webhook):**
* Stripe sends a secret signal to your server: `invoice.payment_succeeded`.
* **Your Server:**
* Updates `Users` table: `plan_status = 'pro'`, `current_period_start = '2025-12-28'`.
* **Generates New Token:** Inserts into `UserTokens` with `usage_count: 0`, `last_updated: '2025-12-28'`.
* **Deletes Old Token:** Removes the free token.





#### Phase 2: Daily Usage (The Worker)

1. **User:** Sends code with API Key.
2. **Worker:** Fetches Token from DB.
3. **Worker:** Checks Date.
* **If Today:** Increment `usage_count`.
* **If Old Date:**
* Calculate Overage (Count - 2000).
* Save to `UsageHistory`.
* Reset `usage_count` to 1.
* Update `last_updated` to Today.




4. **Worker:** Executes Code.

#### Phase 3: The Renewal (Next Month)

1. **Cron Job / Scheduler:** Runs once a day to check for expired subscriptions.
2. **Calculation:**
* Price: $44 (Base).
* Overage: Fetch all items from `UsageHistory` for this month. Sum them up (e.g., 500 extra requests).
* Total: $44 + (500 * $0.0005) = **$44.25**.


3. **Stripe:** You send a command: *"Charge this customer $44.25"*.
4. **Stripe:** Charges card. Sends "Success" Webhook.
5. **Server:** Updates `current_period_start` to next month.

---

### The Plan

We are going to do this one piece at a time.

**Step 1: Database Prep.**
We need to create the `UsageHistory` table and update the `Users` table structure in our minds (add `stripe_customer_id`).

**Step 2: The Stripe Setup.**
We need to install `stripe` and create a "Checkout API" so users can actually click "Buy".

**Step 3: The Webhook.**
This is the most critical part. It listens for Stripe saying "Money Received" and runs your database logic.

--- 
### 1. The Solution to the "Infinite Bill" (The Safety Ceiling)

You are right: Your website is secure. The risk is **User Error**.

* **Scenario:** A student pushes their code to a public GitHub repo with the API key inside. A bot scrapes it and starts mining crypto or running attacks using your API.
* **The Fix:** You don't "block" them at their limit (2,000). You block them at a **Sanity Limit** (Safety Ceiling).

**The Strategy:**

* **Plan Limit:** 2,000 requests (User pays flat fee).
* **Overage Zone:** 2,001 to 20,000 requests (User pays 0.04 per request).
* **Hard Stop (Safety Ceiling):** 20,000 requests.

**Why this works:**

* For a real human developer, 20,000 requests is basically "unlimited." They will likely never hit it.
* For a hacker bot, 20,000 requests stops the attack in minutes.
* **Maximum Loss:** 18,000 extra requests * 0.04 = **720 RS**. You can afford to lose 720 RS. You cannot afford to lose 400,000 RS.

**Action:** We will add a `safety_limit` attribute to the token.

---

### 2. The Final Schema Revision (The Blueprint)

Here is the exact structure we will build. You already have `Users`, `UserTokens`, and `Payments`. We are tweaking them and adding `UsageHistory`.

#### Table 1: `Users` (The Identity)

* **PK:** `email` (String)
* **Attributes:**
* `password`: (Hash)
* `plan_status`: "active" | "inactive" (Modified)
* `stripe_customer_id`: "cus_123..." (**New** - links to Stripe)
* `current_period_start`: "2025-12-28" (**New** - helps calculate monthly bill)



#### Table 2: `UserTokens` (The Worker's Checkpoint)

* **PK:** `token` (String)
* **Attributes:**
* `owner_email`: "user@gmail.com"
* `usage_count`: 1450 (Number)
* `daily_limit`: 2000 (**New** - The Plan Limit)
* `safety_limit`: 20000 (**New** - The Hard Stop)
* `last_updated_date`: "2025-12-28" (**New** - Crucial for the "Lazy Reset")



#### Table 3: `Payments` (The Receipt)

* **PK:** `user_email` (String)
* **SK:** `payment_id` (String)
* **Attributes:**
* `amount`: 3500
* `status`: "succeeded"
* `created_at`: ISO String



#### Table 4: `UsageHistory` (The Overage Ledger - NEW)

* **PK:** `email` (String)
* **SK:** `date` (String, e.g., "2025-12-27")
* **Attributes:**
* `overage_count`: 50 (Number of requests *above* the limit that day)
* `cost`: 2.0 (50 * 0.04)



---

### 3. Step-by-Step Execution

Since you agree to use Stripe (excellent choice), we are doing this:

**Phase 1: Database Setup (Right Now)**
We need to create the `UsageHistory` table and I will give you a script to "migrate" your existing tables (just adding the missing columns conceptually).

**Phase 2: Stripe Integration**
We will install the stripe library and set up the keys.

**Phase 3: The API**
We will build the endpoint that Stripe talks to.

---

### Action: Step 1 - Create the Missing Table

Go to your AWS Console > DynamoDB.

1. **Create Table:**
* **Name:** `UsageHistory`
* **Partition Key:** `email` (String)
* **Sort Key:** `date` (String)
* **Settings:** On-demand capacity.



Once you have done that, **reply "Done"**, and I will give you the Next.js code to install Stripe and set up the checkout flow.
