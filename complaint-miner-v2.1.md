# Skill: complaint-miner

# Complaint Miner v2.1 — Find Validated Product Ideas From Real Pain

Stop guessing what to build. This skill first interviews you to find the right niche, then mines Reddit, App Stores, G2, Upwork, and Capterra for real complaints — clustering them into scored, ranked product opportunities.

Based on the method behind 680+ paying users and $9k/month: find where people are paying for a bad solution, then be less painful.

---

## STEP 0: Niche Discovery (ALWAYS RUN THIS FIRST)

**CRITICAL: Always run Step 0, even if the user gives a topic. The right niche is almost never the first thing someone says.**

Niche discovery has three phases depending on how much the user knows:

---

### Phase A — The User Has No Idea (most common)

If the user says something like "I don't know", "help me find a niche", "no idea", or gives only vague direction — run the full personal interview.

The goal: uncover where the user already has an unfair advantage (lived experience, industry knowledge, personal frustration). The best product ideas come from inside, not from trend lists.

**CRITICAL INTERVIEW RULES:**
1. **Ask ONE question at a time.** Never a list. Never more than one.
2. **Listen to the answer.** Adapt the next question based on what they said.
3. **Dig deeper on signal.** If they mention something interesting, follow up on THAT, don't move to the next scripted question.
4. **Stop when you have enough.** Once you can name 2-3 candidate niches, present them. Don't ask unnecessary questions.

**Opening question — always start here:**
```
What have you done for work the last few years?
```

**Then adapt based on what they say:**

If they name an industry → ask about pain inside it:
```
What's the most frustrating, repetitive, or broken thing you dealt with in [their industry]?
```

If they name a role/job → ask about tools:
```
What tools did you use daily in [their role]? Which ones did you hate?
```

If they mention a specific frustration → dig into it:
```
You mentioned [their frustration]. How did you work around that?
Did you ever look for a solution? What did you find?
```

If they mention something manual/spreadsheet-heavy:
```
Wait — you did [manual thing] by hand? How many hours a week?
Is that something other [people in their role] deal with too?
```

If their answers are still vague → try the frustration angle:
```
Name one tool or software you use that you genuinely hate paying for.
What does it cost and what would you cut from it if you could?
```

Or the community angle:
```
What do people in your circle constantly complain about?
The thing everyone gripes about but nobody has fixed?
```

Or the "I built it myself" angle:
```
Have you ever cobbled together a workaround for something?
Spreadsheets, email templates, copy-pasting between apps?
Tell me about it.
```

**When you have enough signal (usually 2-4 questions in), synthesize:**

Present 3 candidate niches inferred from the conversation:
```
Based on what you've told me, here are 3 niches worth digging into:

1. [NICHE A] — because you said [their exact words]
2. [NICHE B] — because you said [their exact words]
3. [NICHE C] — because you said [their exact words]

Which feels closest to you?
```

Then run a quick Reddit validation on the top candidate (Phase C below) before committing to the full mine.

---

### Phase B — The User Has a Vague Direction

If the user gives a broad space ("restaurants", "freelancers", "e-commerce", "healthcare") but no specific pain — skip the personal interview and go straight to signal discovery.

Run a quick discovery search to surface the top pain clusters in that space:

**If the user gives a broad space** (e.g. "restaurants" or "freelancers"), run a quick discovery search BEFORE the full mine:

```bash
# Run these 3 Reddit searches in parallel to surface pain clusters in the space
# Use the Reddit JSON API directly (not Google — it blocks automated queries)
```

Fetch these three URLs in parallel:
- `https://www.reddit.com/r/{space_subreddit}/search.json?q=frustrated+OR+hate+OR+overkill+OR+%22I+wish%22&sort=top&t=year&limit=10&restrict_sr=1`
- `https://www.reddit.com/r/{space_subreddit}/search.json?q=what+tool+do+you+use+OR+looking+for+alternative&sort=top&t=year&limit=10&restrict_sr=1`
- `https://www.reddit.com/search.json?q={space}+software+frustrated+OR+expensive+OR+%22too+complicated%22&sort=top&t=year&limit=10`

Parse the JSON using Python via Bash:
```bash
python3 - <<'EOF'
import urllib.request, json, urllib.parse, sys

def search(url):
    req = urllib.request.Request(url, headers={"User-Agent": "complaint-miner/2.0"})
    try:
        with urllib.request.urlopen(req, timeout=15) as r:
            data = json.loads(r.read())
        for p in data["data"]["children"][:8]:
            d = p["data"]
            title = d.get("title", "")
            ups = d.get("ups", 0)
            txt = d.get("selftext", "").strip()[:200]
            print(f"[{ups} ups] r/{d.get('subreddit')} — {title}")
            if txt: print(f"  → {txt}")
    except Exception as e:
        print(f"Error: {e}")

# Replace SPACE and SUBREDDIT with actual values
search("https://www.reddit.com/r/SUBREDDIT/search.json?q=frustrated+OR+hate+OR+overkill&sort=top&t=year&limit=10&restrict_sr=1")
EOF
```

Present the top pain clusters found and ask:
```
Here are the biggest complaint clusters I found in [SPACE]:

1. [Pain cluster A] — N mentions
2. [Pain cluster B] — N mentions
3. [Pain cluster C] — N mentions

Which one do you want to dig into? Or pick a different angle.
```

---

### Phase C — The User Has a Specific Niche

If the user provides a specific niche (e.g. "freelance invoicing tools", "restaurant scheduling"), skip the interview and go directly to Step 1.

---

## STEP 1: Confirm Niche + Announce

Once niche is confirmed, display:

```
Mining complaints for: {NICHE}

Sources:
  ✓ Reddit (primary — real people, no PR filter)
  ✓ Upwork (money signal — people already paying to solve this)
  ✓ Trustpilot + App Store reviews (structured anger)
  ✓ G2 / Capterra (SaaS-specific pain, with fallbacks)

Starting all searches in parallel...
```

Store:
- `NICHE` = confirmed niche
- `SUBREDDITS` = 2-3 most relevant subreddits for this niche (infer from niche)

**Subreddit mapping guide:**
- freelancers → r/freelance, r/Upwork, r/selfemployed
- small business → r/smallbusiness, r/Entrepreneur, r/BusinessImprovement
- restaurants / hospitality → r/KitchenConfidential, r/restaurant, r/smallbusiness
- software/SaaS → r/SaaS, r/Entrepreneur, r/startups
- creators / solopreneurs → r/solopreneur, r/SideProject, r/Entrepreneur
- e-commerce → r/ecommerce, r/Etsy, r/shopify
- property management → r/PropertyManagement, r/landlord
- healthcare → r/medicine, r/medicalschool, r/nursing
- trades → r/Construction, r/Plumbing, r/electricians

---

## STEP 2: Run All Searches in Parallel

**CRITICAL: Run ALL of these as parallel tool calls in a single message.**

### A. Reddit (PRIMARY — always use JSON API directly, never Google)

For each of the 2-3 identified subreddits, run:

```bash
python3 - <<'EOF'
import urllib.request, json, urllib.parse

NICHE = "{NICHE}"  # fill in
SUBREDDITS = ["{SUB1}", "{SUB2}", "{SUB3}"]  # fill in
QUERIES = [
    "frustrated OR hate OR annoying",
    "overkill OR \"too expensive\" OR \"too complicated\"",
    "\"wish there was\" OR \"looking for alternative\" OR \"why is there no\"",
    "\"I just want\" OR \"all I need\" OR \"simple version\"",
    "switched OR alternative OR replacement",
]

def search(subreddit, query, niche):
    q = urllib.parse.quote(f"{niche} {query}")
    url = f"https://www.reddit.com/r/{subreddit}/search.json?q={q}&sort=top&t=all&limit=8&restrict_sr=1"
    req = urllib.request.Request(url, headers={"User-Agent": "complaint-miner/2.0"})
    try:
        with urllib.request.urlopen(req, timeout=12) as r:
            data = json.loads(r.read())
        results = []
        for p in data["data"]["children"]:
            d = p["data"]
            results.append({
                "ups": d.get("ups", 0),
                "subreddit": d.get("subreddit"),
                "title": d.get("title", ""),
                "text": d.get("selftext", "").strip()[:300],
                "url": d.get("url", ""),
            })
        return results
    except Exception as e:
        return [{"error": str(e)}]

all_results = []
for sub in SUBREDDITS:
    for q in QUERIES[:3]:  # top 3 query patterns
        results = search(sub, q, NICHE)
        all_results.extend(results)

# Deduplicate by title, sort by upvotes
seen = set()
unique = []
for r in sorted(all_results, key=lambda x: x.get("ups", 0), reverse=True):
    if r.get("title") and r["title"] not in seen:
        seen.add(r["title"])
        unique.append(r)

for r in unique[:15]:
    if "error" in r:
        print(f"[ERROR] {r['error']}")
        continue
    print(f"\n[{r['ups']} ups] r/{r['subreddit']} — {r['title']}")
    if r['text']: print(f"  → {r['text']}")
EOF
```

### B. Upwork (money-already-being-spent signal)

Fetch this URL directly and parse the job titles:
```
https://www.upwork.com/search/jobs/?q={NICHE}&sort=recency
```

If Upwork blocks (403), fallback to:
```
https://www.reddit.com/search.json?q={NICHE}+site:upwork.com+OR+fiverr+job&sort=top&t=year&limit=10
```

Look for: recurring job titles (same job posted by many clients = product opportunity).

### C. Trustpilot (fallback for G2/Capterra which often block)

Search for the top 2-3 tools in this niche on Trustpilot:

```bash
python3 - <<'EOF'
import urllib.request, json, urllib.parse

NICHE = "{NICHE}"

# First find the dominant tools in this niche
url = f"https://www.reddit.com/search.json?q={urllib.parse.quote(NICHE)}+what+tool+do+you+use+OR+recommendation&sort=top&t=all&limit=5"
req = urllib.request.Request(url, headers={"User-Agent": "complaint-miner/2.0"})
with urllib.request.urlopen(req, timeout=12) as r:
    data = json.loads(r.read())
for p in data["data"]["children"][:5]:
    d = p["data"]
    print(f"[{d.get('ups',0)} ups] {d.get('title','')}")
    print(f"  → {d.get('selftext','').strip()[:200]}")
EOF
```

Then for each dominant tool identified, fetch its Trustpilot reviews page:
`https://www.trustpilot.com/review/{tool-name}.com?stars=1&stars=2`

Parse the review text for complaint patterns.

**G2 fallback:** If `site:g2.com` is blocked, try fetching a specific product page:
`https://www.g2.com/products/{product-slug}/reviews?filters[review_type]=all&filters[sort]=most_helpful`

**Capterra fallback:** Try:
`https://www.capterra.com/p/{product-id}/{product-name}/reviews/`

If both block, note it in the report and rely on Reddit + Trustpilot.

### D. App Store signal

Search Reddit for app store complaints:
```
https://www.reddit.com/search.json?q={NICHE}+app+"1+star"+OR+"worst+app"+OR+"deleted+it"&sort=top&t=year&limit=8
```

---

## STEP 3: Fetch Top Threads

For the top 3-5 Reddit threads found (highest upvotes, most relevant titles), fetch the actual thread to read top comments:

```bash
python3 - <<'EOF'
import urllib.request, json

THREAD_URL = "{REDDIT_THREAD_URL}.json?sort=top&limit=10"
req = urllib.request.Request(THREAD_URL, headers={"User-Agent": "complaint-miner/2.0"})
with urllib.request.urlopen(req, timeout=12) as r:
    data = json.loads(r.read())

# Post body
post = data[0]["data"]["children"][0]["data"]
print(f"POST: {post.get('title')}")
print(f"BODY: {post.get('selftext','')[:500]}")
print()
print("TOP COMMENTS:")
for c in data[1]["data"]["children"][:8]:
    d = c["data"]
    if d.get("body") and d.get("body") != "[deleted]":
        print(f"  [{d.get('ups',0)} ups] {d.get('body','')[:300]}")
EOF
```

**Why comments matter:** The post title is the question. The top comments are where the real pain lives — specific tool names, specific frustrations, specific workarounds people have cobbled together.

---

## STEP 4: Extract and Cluster Complaints

Read all data collected. Extract every distinct complaint. Group into clusters.

**Cluster format:**
```
[PAIN CLUSTER NAME]
- Complaint examples (verbatim quotes preferred)
- Source count: N Reddit threads, N Trustpilot reviews, N comments
- Signal type: frustrated-user | paying-for-bad-solution | recurring-job | overkill-signal
- Evidence of willingness to pay: yes (paying $X/mo for bad solution) | partial | no
```

**Signal types:**
- **frustrated-user** — complaints about existing tools
- **paying-for-bad-solution** — person pays $X/month for something that half-works
- **recurring-job** — same Upwork/Fiverr job posted by many different clients = automation gap
- **overkill-signal** — "this tool does way too much, I just need X" = underserved simple use case

---

## STEP 5: Score and Rank Opportunities

Score each cluster on three dimensions (1–5 each):

| Dimension | What it means |
|-----------|---------------|
| **Pain intensity** | How angry/frustrated are people? 1=mild, 5=blocking work |
| **Market size** | How many people have this problem? 1=tiny niche, 5=millions |
| **Competition quality** | How bad are existing solutions? 1=great tools exist, 5=nothing works |

**Opportunity Score = Pain × Market × Competition (max 125)**

Sort highest to lowest. Report top 5.

---

## STEP 6: Output the Report

```
═══════════════════════════════════════════════════════
  COMPLAINT MINER REPORT — {NICHE}
  Sources: Reddit ({N} threads), Upwork, Trustpilot
  Searches run: {N} | Complaints extracted: {N}
═══════════════════════════════════════════════════════

## TOP OPPORTUNITIES (ranked by score)

### #1 — [OPPORTUNITY NAME] (Score: XX/125)
Pain: X/5 | Market: X/5 | Competition: X/5

What people say:
> "[verbatim quote]" — r/subreddit, N upvotes
> "[verbatim quote]" — Trustpilot, 1★
> "[job title]" — Upwork (recurring posting)

Signal: [overkill-signal | paying-for-bad-solution | frustrated-user | recurring-job]
Willingness to pay: [Yes — already paying $X/mo for bad solution | Upwork postings at $Y each]

What to build: [1-2 sentence concept. Be specific — not "an invoicing tool" but
  "invoicing-only for freelancers who just left QuickBooks — send, track, get paid, nothing else."]

Validation step: [exact action to take this week]

---

### #2 — [OPPORTUNITY NAME] (Score: XX/125)
[same format]

[...top 5 total]

═══════════════════════════════════════════════════════
## THE OVERKILL SIGNAL
Tools called "overkill" or "too complex" in this niche:
- [Tool name] — "[exact quote]" (N mentions)
  → Simple version gap: [what people say they actually need]

## SOURCES SEARCHED
- Reddit threads read: N
- Trustpilot reviews: N (tool: X) | N (tool: Y)
- Upwork postings found: N
- G2/Capterra: [fetched / blocked — fallback used]

## VALIDATION CHECKLIST
Before building anything from this report:
☐ Find 10 real people who have this problem (use the subreddits above)
☐ Ask: "Do you currently pay anything to solve this?" (tool/freelancer/manual)
☐ Test your one-liner: describe solution in one sentence
   → If they ask "where do I sign up?" you have a product
☐ Charge before you build — presell or require a deposit to join waitlist
═══════════════════════════════════════════════════════
```

---

## Step 7: Validate + Post

After delivering the report, offer to validate the #1 opportunity by drafting a Reddit post. Ask the user:

 permission to post it a specific subreddit. If they decline, just them to know the target subreddit and offer to dig deeper instead.

| Signal | `post for validation` | `post as validation` | `post as validation` | ask one question. Wait for answer. Then:

**Step 8: Draft the Validation Post**

Based on the user's answer, draft the exact post text for the most relevant subreddit. Choose the subreddit based on where the complaints were densest (from the report, —r/smallbusiness, r/Bookkeeping, r/Entrepreneur, r/selfemployed).

 Use the template that matches the subreddit's topic. Keep it genuine — ask a genuine question, not a funnel.

 ads.

 Don't say "I'm building an app" or mention "revenue if you ask how you'd pay. Include a clear CTA if relevant.

 Example below — adapt to your niche:

---

**Validation post template:**

```
### Option A — For existing niche, complainer
"I'm building a [simple version of [overkill tool]. What do people say they actually need]

**Title:****
"[Product concept] — would this solve [pain #1]?

I'm a solo [role] — [freelancer / small business owner / developer]
and I've been thinking about building an AI that auto-categorizes my credit card transactions from receipt photos.
  Here's what I've built so far.  Would you pay for this?
```

### Option B — For people new to a niche
```
I'm looking at [specific role] — describe tool in one sentence, ideally eliminates as much [overkill] as possible."
```

### Option C — For people overwhelmed by complexity
```
"I just need something simple that does what [X product]. I'd pay $Y/month for something that does 90% of what I need. Such features I shouldn't need."
```

---

**How to post:**
1. Choose the most relevant subreddit (based on where complaints clustered)
2. Draft the post text using the template above
3. Show the user the post for approval BEFORE posting
 If they confirm, POST IT
4. If they decline, note which they want to test and offer to dig deeper into the niche instead

 Show the post text and say "Here's a draft — feel free to adjust it if needed:"

---

After posting or report back with any replies. Count upvotes, comment sentiment, and flag users who say "take my money" — these are early adopters.

 Track them over 1-2 weeks and revisit the post to see if interest is growing.

 run the full mine on that specific sub-niche.

 If not, move to the next opportunity.

```
I've mined {N} complaints across {M} sources for: {NICHE}

Next steps:
- Dig into sub-niche: "{NICHE} for [specific audience]"
- Read the top Reddit thread in full for more verbatim complaints
- Run the full mine on [sub-niche]
- Just tell me what you want next.
```

---

*Complaint Miner v2.1 — built by Clawver (@clawvershipps on X)*
*Part of the Clawver Skill Pack — tools for autonomous builders*
