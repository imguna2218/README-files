# Claude Code Usage Guide for IgnisOps

## 1. What Are Tokens? (And Why Should You Care?)

Every interaction with Claude consumes **tokens**. A token is roughly a word or a part of a word (e.g., `"IgnisOps"` is one token, `"microVM orchestration"` is ~3 tokens). Both your **questions (input)** and Claude’s **answers (output)** count toward the total.

- **Input tokens**: Everything you type, plus any files or conversation history Claude has to read.
- **Output tokens**: The code, explanations, or suggestions Claude generates.

If you run out of tokens (e.g., monthly limit), you can’t continue until the next cycle. **Efficient usage means you can build more features for the same token budget.**

---

## 2. Token Utilization: Good vs. Bad Practices

### ❌ Bad: Pasting Entire READMEs Every Time

```bash
# DON'T do this
I want you to implement the entire vessel-frontend dashboard based on this README:
[pastes 5000 words of vessel-frontend.md]
```

- **Why it’s bad**: You’re re‑uploading the same huge context every conversation. If you work on multiple features, you waste tokens repeating the same information.

### ✅ Good: Use `@` References + Targeted Instructions

Claude Code can read files directly from your repo using `@filename`. This still consumes tokens for the file content, but you do it **once per conversation** and then refer to specific parts.

```bash
# DO this
@docs/vessel-frontend.md – look at the "Dashboard" section and implement the project list component.
```

- **Why it’s good**: Claude reads only what’s necessary for the current task. You avoid pasting, and the file content is loaded efficiently.

### 🚀 Best: Combine `@` with a Project‑Level `CLAUDE.md`

Create a `CLAUDE.md` file in your repo that summarises the project structure and key documents. Then use it as a map.

```markdown
# CLAUDE.md – IgnisOps Project Context

## Modules & Docs
- **vessel-frontend**: UI for project management. See `/docs/vessel-frontend.md` for specs.
- **vessel-backend**: Orchestration API. See `/docs/vessel-backend.md` for endpoints.
- **vessel-infra**: VM lifecycle. See `/docs/vessel-infra.md`.
- **vessel-agent**: AI service. See `/docs/vessel-agent.md`.

## Coding Standards
- Use TypeScript with strict mode.
- Follow the existing folder structure.
```

Now in a conversation:

```bash
@CLAUDE.md – I need to add a delete button to the project card in vessel-frontend. Read only the frontend doc and implement it.
```

- **Why it’s best**: Claude loads the lightweight `CLAUDE.md` first, then you direct it to specific docs. No redundant content, and you always have a shared context.

---

## 3. Prompt Engineering: Organized vs. Unorganized

Your prompt shapes Claude’s output. A vague prompt leads to hallucinations or irrelevant code. A well‑structured prompt saves tokens and yields accurate results.

### ❌ Unorganized Prompt (Token‑Wasting & Risky)

```bash
Make the frontend better. Also fix the backend. Oh and add AI.
```

- **Problem**: Too vague. Claude will guess, likely generate wrong code, and you’ll need multiple corrections – wasting tokens.

### ✅ Organized Prompt (Precise & Efficient)

```bash
@docs/vessel-frontend.md

Task: Implement the template selection modal as described in the "Template Selection Modal" section.

Requirements:
- Use React with Tailwind CSS (already set up).
- Modal should open when user clicks "New Project".
- Show template cards with name, description, icon.
- On card click, call `/api/projects/create` with the selected template ID.

Output only the component code and any necessary imports.
```

- **Why it’s good**: You tell Claude exactly which file to read, what part to focus on, the tech stack, the expected behavior, and what to output. This reduces back‑and‑forth and token waste.

### 🧠 Prompt Structure Template

```
@<relevant-doc.md>

Task: <one-sentence summary>

Details:
- <bullet points of specific requirements>
- <any constraints (e.g., "use existing utils", "follow the pattern in file X")>

Output: <what you want: e.g., "only the component code", "explain the changes first, then code">
```

---

## 4. Phased Development: Build Features Step by Step

Instead of asking Claude to build an entire feature in one go, break it into **phases**. This reduces errors, makes debugging easier, and keeps token usage predictable.

### Example: Building the Project Dashboard

#### Phase 1 – Skeleton & Data Fetching
```bash
@docs/vessel-frontend.md

Phase 1 of dashboard:
- Create a `Dashboard` component that fetches the list of projects from `/api/projects/list` on mount.
- Show a loading spinner while fetching.
- Display a simple list of project names (no styling yet).
- Use the existing API client in `services/api.ts`.

Output the component code and any changes needed.
```

#### Phase 2 – Project Cards & UI
```bash
@docs/vessel-frontend.md

Phase 2:
- Replace the plain list with project cards (use the design from the doc).
- Each card should show: name, stack icon, last opened date.
- Add an "Open" button that calls the open project API.
- Keep the existing fetch logic from Phase 1.

Output the updated component.
```

#### Phase 3 – Interactivity & Modals
```bash
@docs/vessel-frontend.md

Phase 3:
- Add a delete icon to each card.
- When clicked, show a confirmation modal (use the existing modal component).
- On confirm, call the delete API and remove the card from the list without refreshing.

Output the new code.
```

- **Why it works**: Each phase builds on the previous one. Claude works with a smaller context, you can verify each step, and if something breaks, you know exactly which phase introduced the issue. Token usage stays low because you're only adding incremental instructions.

### Example: Implementing a Backend Endpoint

#### Phase 1 – Route & Basic Handler
```bash
@docs/vessel-backend.md

Phase 1 for `/api/projects/delete`:
- Add a new DELETE route in `routes/projects.js`.
- It should accept `project_id` and `user_id` from the authenticated user (assume `req.user` is set).
- Return a 202 Accepted immediately (we'll add the actual deletion later).
- Log the request.

Output the route code.
```

#### Phase 2 – Integrate with Infra & Supabase
```bash
@docs/vessel-backend.md

Phase 2:
- In the delete route, call `vessel-infra` to destroy the VM (use the existing infra client).
- Then archive the project in Supabase (use `archiveProject(project_id)` from `storage.js`).
- Handle errors and return appropriate status codes.

Output the updated route.
```

---

## 5. Summary: Best Practices at a Glance

| What to Do | Why |
|------------|-----|
| **Use `@` to reference docs** instead of pasting them. | Saves tokens, avoids repetition, and keeps context fresh. |
| **Keep a `CLAUDE.md`** with project overview and doc links. | Gives Claude a permanent lightweight context across conversations. |
| **Write structured prompts** – include doc reference, task summary, bullet points, and output format. | Reduces hallucination and back‑and‑forth. |
| **Break features into phases** (3‑5 steps per phase). | Lower token cost per interaction, easier debugging, and better control. |
| **Start a new conversation for each major feature or phase.** | Prevents conversation history from bloating and eating tokens. |
| **Monitor your token usage** (Claude shows it per conversation). | Helps you adjust your approach if needed. |

---

## 6. Example End‑to‑End Workflow for a New Feature

Let’s say you need to add **project search** to the dashboard.

1. **New conversation** (no history).
2. **Reference the doc**:
   ```
   @docs/vessel-frontend.md
   ```
3. **Phase 1 prompt**:
   ```
   Phase 1 of search feature:
   - Add a search input above the project grid.
   - On typing, filter the already‑loaded project list by name (client‑side).
   - No API calls yet.
   Output the updated Dashboard component.
   ```
4. **Review output**, test, and accept.
5. **Phase 2 prompt** (same conversation):
   ```
   Phase 2:
   - Replace client‑side filtering with a backend search.
   - Call `/api/projects/search?q=...` (we'll implement that next).
   - Show a "Searching..." indicator.
   Output the changes.
   ```
6. **Phase 3** (maybe new conversation for backend part):
   ```
   @docs/vessel-backend.md
   Implement `/api/projects/search`:
   - Accept `q` query param.
   - Query Supabase projects table with ILIKE.
   - Return matching projects.
   Output the route code.
   ```

By the end, you’ve built the feature with minimal token waste and high confidence.

---

**Now go build IgnisOps like a pro!** If you ever feel stuck or notice Claude going off‑track, refine your prompt or break the task into even smaller pieces. Happy coding!
