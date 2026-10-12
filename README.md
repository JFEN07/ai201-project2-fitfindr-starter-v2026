# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

**FitFindr** is an agent that aids in finding reasonable, complementing thrifts through a quick lookup. For example if someone asks for "a vintage graphic tee under $30, size M". The agent will take that input, use it to search against its listings and return items that match it, determines other items that would match with it, and writes a caption for it. This agent is also equipped with multiple tools listed below and acts by using different tools in different orders given the request and what is returned.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Finds clothing listings through keyword overlap, optionally filtering by size and max price, alongside case-insensitive size token matching so "M" matches "S/M", but "L" doesn't match "XL".
- **Inputs:** description: str, size: str | None = None, max_price: float | None = None<!-- name and type each: `max_price` (float), not "a price" -->Description is required, while size and max price are not.
- **Returns:** A list[dict] of matching listings ordered by keyword-overlap score, highest first. Each dictionary contains id, title, description, category, style_tags, size, condition, price, colors, brand, and platform, with brand possibly None.
- **When it has nothing:** Returns an empty list when nothing matches — an empty list, not None, and not an exception.

### `suggest_outfit`

- **What it does:** Given a thrifted item and the user's wardrobe, this calls the model to suggest one or two outfits.
- **Inputs:** new_item: dict, containing a listing record; wardrobe: dict, containing an items key whose value is a list of wardrobe-item dictionaries.
- **Returns:** A non-empty string with outfit suggestions.
- **When it has nothing:** Returns general styling advice rather than raising or returning "".

### `create_fit_card`

- **What it does:** Uses the model to write a short caption about the selected item and suggested outfit.
- **Inputs:** outfit: str, containing the outfit suggestion; new_item: dict, containing the selected listing record.
- **Returns:** A str containing a two-to-four-sentence caption that mentions the item, its price, its platform once each, and it describes the outfit’s vibe. It should return a slightly different caption even if the exact same item is selected and suggested outfit is produced.
- **When it has nothing:** Returns a descriptive message.
---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put a helpful message in session["error"] and stop without calling suggest_outfit or create_fit_card. Otherwise, store the first result in session["selected_item"] and pass that item and session["wardrobe"] to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The parse_query uses regular expressions, without calling the model, to extract a size and maximum price. It removes the matched portions from the query and uses the remaining text as the description. An unrecognized or unspecified size or price becomes None.

**What moves through the session:** The session starts with the original query and the supplied wardrobe. The extracted description, size, and maximum price are stored in parsed, followed by the matching listings in search_results. If no listings match, a message is stored in error and the run stops. Otherwise, the first result is stored in selected_item, which is passed with wardrobe to the outfit tool. Its response is stored in outfit_suggestion, then passed with the selected item to the fit-card tool. The resulting caption is stored in fit_card, and the completed session is returned. If a model call raises ModelUnavailable, the agent records a message in error and returns the session with the results collected so far.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

```


**The three tools, tested one at a time**

*search_listings*
```
python -c "import json; from tools import search_listings; print(json.dumps(search_listings('graphic tee', max_price=30), indent=2, ensure_ascii=False))"
[
  {
    "id": "lst_002",
    "title": "Y2K Baby Tee — Butterfly Print",
    "description": "Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.",
    "category": "tops",
    "style_tags": [
      "y2k",
      "vintage",
      "graphic tee",
      "cottagecore"
    ],
    "size": "S/M",
    "condition": "excellent",
    "price": 18.0,
    "colors": [
      "white",
      "pink",
      "purple"
    ],
    "brand": null,
    "platform": "depop"
  },
  {
    "id": "lst_006",
    "title": "Graphic Tee — 2003 Tour Bootleg Style",
    "description": "Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.",
    "category": "tops",
    "style_tags": [
      "graphic tee",
      "vintage",
      "grunge",
      "streetwear",
      "band tee"
    ],
    "size": "L",
    "condition": "good",
    "price": 24.0,
    "colors": [
      "black"
    ],
    "brand": null,
    "platform": "depop"
  },
  {
    "id": "lst_017",
    "title": "Mesh Long-Sleeve Top — Black",
    "description": "Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.",
    "category": "tops",
    "style_tags": [
      "y2k",
      "grunge",
      "goth",
      "layering"
    ],
    "size": "S/M",
    "condition": "excellent",
    "price": 15.0,
    "colors": [
      "black"
    ],
    "brand": null,
    "platform": "depop"
  },
  {
    "id": "lst_033",
    "title": "Vintage Band Tee — Faded Grey",
    "description": "Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.",
    "category": "tops",
    "style_tags": [
      "vintage",
      "grunge",
      "band tee",
      "graphic tee",
      "streetwear"
    ],
    "size": "L",
    "condition": "fair",
    "price": 19.0,
    "colors": [
      "grey",
      "charcoal"
    ],
    "brand": null,
    "platform": "depop"
  },
  {
    "id": "lst_011",
    "title": "Low-Rise Cargo Pants — Khaki",
    "description": "Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.",
    "category": "bottoms",
    "style_tags": [
      "y2k",
      "cargo",
      "2000s",
      "streetwear"
    ],
    "size": "W29",
    "condition": "fair",
    "price": 27.0,
    "colors": [
      "khaki",
      "tan"
    ],
    "brand": null,
    "platform": "poshmark"
  },
  {
    "id": "lst_015",
    "title": "Vintage Graphic Hoodie — Faded Black",
    "description": "Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.",
    "category": "tops",
    "style_tags": [
      "vintage",
      "grunge",
      "graphic",
      "streetwear"
    ],
    "size": "L",
    "condition": "fair",
    "price": 26.0,
    "colors": [
      "black",
      "charcoal"
    ],
    "brand": null,
    "platform": "depop"
  }
]

```

*search_listings empty case*
```
python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=-1))"
[]
```

*suggest_outfit*
```
python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Outfit 1: Casual Streetwear
- New Item: Vintage Levi's 501 Jeans (Medium Wash)
- Wardrobe Pieces: White ribbed tank top (w_003), Oversized grey crewneck sweatshirt (w_004), Chunky white sneakers (w_007), Black crossbody bag (w_010)
- Why it works: This is an effortless, balanced everyday look. The fitted white tank contrasts nicely with the roomier fit of the 501s. Layering the oversized grey crewneck on top creates a classic, comfortable streetwear silhouette, while the chunky white sneakers and black crossbody bag tie the casual aesthetic together.

Outfit 2: Edgy Vintage
- New Item: Vintage Levi's 501 Jeans (Medium Wash)
- Wardrobe Pieces: Black cropped zip hoodie (w_005), Vintage black denim jacket (w_006), Black combat boots (w_008), Brown leather belt (w_009)
- Why it works: This combination plays on proportions and textures. Tucking in a top with the brown leather belt highlights the waist of the classic 501s. Pairing a black cropped zip hoodie under a slightly cropped black denim jacket creates a cool, layered top half that is grounded by the edgy black combat boots.
```

*suggest_outfit empty case*
```
python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
Style these vintage Levi's 501s with a tucked-in white ribbed tank top or a vintage band tee for a classic, effortless look. Layer with an oversized black leather biker jacket or a brown suede blazer. Complete the outfit with retro sneakers like Adidas Sambas or black leather loafers, and accessorize with a simple black leather belt and silver hoop earrings.
```

*create_fit_card*
```
python -c "from tools import create_fit_card; from utils.data_loader import load_listings; item = load_listings()[0]; [print(f'\nRun {i + 1}:\n', create_fit_card('jeans and white sneakers', item)) for i in range(3)]"

Run 1:
 Found these classic Vintage Levi's 501 Jeans and they are officially my new favorite pair. They're listed on Depop for just $38.00 and have the best lived-in look. Throw them on with some fresh white sneakers for the ultimate effortless, everyday vibe.

Run 2:
 Just listed these vintage Levi's 501 jeans on Depop for $38.0, and they have the best broken-in medium wash with light knee fading. Throw them on with some fresh white sneakers for that effortlessly cool, everyday streetwear vibe. Grab them before they're gone!

Run 3:
 Scored these vintage Levi's 501 jeans on Depop for just $38, and I am obsessed with the medium wash and knee fading. I'm styling them with crisp white sneakers for that effortlessly cool, casual weekend look. Grab them before I change my mind!
```

*create_fit_card empty case*
```
python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('', load_listings()[0]))"
Cannot create a fit card because no outfit suggestion was provided.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
