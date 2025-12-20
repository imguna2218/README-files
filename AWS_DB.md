You are taking the right step. Before you charge anyone or run any code, you need a place to store your users.

**No, you do not need to deploy the website to use DynamoDB.**
You can (and should) connect to the **real** AWS DynamoDB from your **local** laptop (`localhost:3000`) while developing. This works perfectly as long as you have internet access.

Here is the exact, step-by-step guide to building the **Authentication System (Signup/Login)** with Next.js and DynamoDB.

### Phase 1: Create the Database (AWS Console)

We will create a table to store user credentials (`email`, `password_hash`) and their API profile.

1. **Log in** to the AWS Console.
2. Search for **DynamoDB** and click it.
3. Click **"Create table"** (Orange button).
4. **Table details:**
* **Table name:** `Users`
* **Partition key:** `email` (String)
* **Sort key:** Leave blank.


5. **Table settings:**
* Select **"Customize settings"**.
* **Capacity mode:** Select **On-demand** (This is crucial. It means you pay $0.00 when idle and only pay per request. Perfect for startups).


6. Click **"Create table"**.

### Phase 2: Setup Next.js Environment

You need to allow your Next.js app to talk to AWS.

1. **Install Dependencies:**
Open your Next.js terminal and run:
```bash
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb bcryptjs jsonwebtoken cookie

```


* `@aws-sdk`: To talk to DynamoDB.
* `bcryptjs`: To encrypt passwords (NEVER store plain passwords).
* `jsonwebtoken`: To create a session cookie so users stay logged in.


2. **Environment Variables:**
Open (or create) `.env.local` in your Next.js root. Add your AWS keys:
```env
NEXT_PUBLIC_AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key_here
AWS_SECRET_ACCESS_KEY=your_secret_key_here
JWT_SECRET=some_super_secret_random_string

```



### Phase 3: Create the Database Client

Create a helper file so we don't repeat code.

**File:** `lib/dynamo.js` (Create this folder/file if needed)

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({
  region: process.env.NEXT_PUBLIC_AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
});

// The "DocumentClient" makes it easier to work with JSON
const db = DynamoDBDocumentClient.from(client);

export { db };

```

### Phase 4: Implement Signup (The Registration API)

This API will check if the user exists, hash their password, and save them.

**File:** `pages/api/auth/signup.js`

```javascript
import { db } from "../../../lib/dynamo";
import { GetCommand, PutCommand } from "@aws-sdk/lib-dynamodb";
import bcrypt from "bcryptjs";
import { v4 as uuidv4 } from "uuid"; // Run 'npm install uuid' if you haven't

export default async function handler(req, res) {
  if (req.method !== "POST") return res.status(405).end();

  const { email, password } = req.body;

  try {
    // 1. Check if user already exists
    const checkUser = await db.send(new GetCommand({
      TableName: "Users",
      Key: { email },
    }));

    if (checkUser.Item) {
      return res.status(400).json({ error: "User already exists" });
    }

    // 2. Hash the password (Security Critical)
    const hashedPassword = await bcrypt.hash(password, 10);

    // 3. Generate their initial API Token (for later use)
    const initialToken = uuidv4();

    // 4. Save to DynamoDB
    const newUser = {
      email,
      password: hashedPassword,
      api_token: initialToken,
      plan: "free",
      created_at: new Date().toISOString(),
    };

    await db.send(new PutCommand({
      TableName: "Users",
      Item: newUser,
    }));

    // 5. Success
    return res.status(201).json({ message: "User created!", token: initialToken });

  } catch (error) {
    console.error(error);
    return res.status(500).json({ error: "Internal Server Error" });
  }
}

```

### Phase 5: Implement Login (The Sign-In API)

This API verifies the password and sets a cookie.

**File:** `pages/api/auth/login.js`

```javascript
import { db } from "../../../lib/dynamo";
import { GetCommand } from "@aws-sdk/lib-dynamodb";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import { serialize } from "cookie";

export default async function handler(req, res) {
  if (req.method !== "POST") return res.status(405).end();

  const { email, password } = req.body;

  try {
    // 1. Fetch User
    const { Item: user } = await db.send(new GetCommand({
      TableName: "Users",
      Key: { email },
    }));

    if (!user) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // 2. Verify Password
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // 3. Create Session Token (JWT)
    const token = jwt.sign(
      { email: user.email, plan: user.plan, api_token: user.api_token },
      process.env.JWT_SECRET,
      { expiresIn: "1d" }
    );

    // 4. Set Cookie (HttpOnly - Secure)
    res.setHeader("Set-Cookie", serialize("auth_token", token, {
      httpOnly: true,
      secure: process.env.NODE_ENV !== "development",
      maxAge: 60 * 60 * 24, // 1 day
      sameSite: "strict",
      path: "/",
    }));

    return res.status(200).json({ 
        message: "Login successful", 
        api_token: user.api_token // Give them their API key so they can use it!
    });

  } catch (error) {
    console.error(error);
    return res.status(500).json({ error: "Internal Server Error" });
  }
}

```

### Phase 6: How to Connect this to your Frontend

You don't need complex logic. Just standard fetch calls.

**Example `pages/signup.js` Snippet:**

```javascript
async function handleSignup(e) {
  e.preventDefault();
  const res = await fetch("/api/auth/signup", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password }),
  });
  
  const data = await res.json();
  if (res.ok) {
    alert("Account created! Please login.");
  } else {
    alert("Error: " + data.error);
  }
}

```

### Summary of What You Just Built

1. **Database:** A generic `Users` table in DynamoDB (Cost: $0.00).
2. **Signup:** Creates a user, encrypts their password, and **auto-generates the `api_token**` they will use for your EvalX service.
3. **Login:** Verifies the user and gives them their `api_token` back.

**Next Step:**
Implement these files. Once you can log in via your website on `localhost:3000`, verify that you can see the new item in the AWS DynamoDB Console under the "Explore items" tab.
