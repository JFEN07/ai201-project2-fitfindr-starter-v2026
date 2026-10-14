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
python agent.py
=== A query the data can match ===
[1] parse_query
      in:  looking for a vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
=== Item received by suggest_outfit ===
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
}
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Here are two outfit suggestions using the Y2K Baby Tee with Butterfly Print and pieces from your wardrobe:  **…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: I just scored this adorable Y2K Baby Tee with Butterfly Print on Depop for only $18, and it's giving major nos…
=== Complete session ===
{
  "query": "looking for a vintage graphic tee under $30",
  "parsed": {
    "description": "looking for a vintage graphic tee",
    "size": null,
    "max_price": 30.0
  },
  "search_results": [
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
      "id": "lst_003",
      "title": "Oversized Flannel Shirt — Plaid Red/Black",
      "description": "Classic oversized flannel. Great layering piece. A few tiny pulls in the fabric but nothing visible when worn.",
      "category": "tops",
      "style_tags": [
        "grunge",
        "vintage",
        "flannel",
        "streetwear",
        "layering"
      ],
      "size": "XL (oversized)",
      "condition": "good",
      "price": 22.0,
      "colors": [
        "red",
        "black"
      ],
      "brand": "Woolrich",
      "platform": "thredUp"
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
      "id": "lst_012",
      "title": "Oversized Crewneck Sweatshirt — Vintage Navy",
      "description": "Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.",
      "category": "tops",
      "style_tags": [
        "vintage",
        "basics",
        "oversized",
        "classic"
      ],
      "size": "XL (fits oversized)",
      "condition": "good",
      "price": 20.0,
      "colors": [
        "navy"
      ],
      "brand": null,
      "platform": "thredUp"
    },
    {
      "id": "lst_013",
      "title": "90s Silk Slip Dress — Floral, Midi Length",
      "description": "Delicate 90s slip dress in a muted floral print. Midi length, adjustable straps. Light snag on the side seam — not visible when worn.",
      "category": "bottoms",
      "style_tags": [
        "90s",
        "vintage",
        "feminine",
        "floral",
        "cottagecore"
      ],
      "size": "M",
      "condition": "good",
      "price": 30.0,
      "colors": [
        "ivory",
        "dusty pink",
        "green"
      ],
      "brand": null,
      "platform": "depop"
    },
    {
      "id": "lst_014",
      "title": "Leather Belt — Brown, Braided",
      "description": "Genuine leather braided belt. Adjustable, multiple holes. Classic Western buckle. Can be dressed up or down.",
      "category": "accessories",
      "style_tags": [
        "vintage",
        "western",
        "classic",
        "earth tones"
      ],
      "size": "One Size (adjustable)",
      "condition": "excellent",
      "price": 12.0,
      "colors": [
        "brown"
      ],
      "brand": null,
      "platform": "thredUp"
    }
  ],
  "selected_item": {
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
  "wardrobe": {
    "items": [
      {
        "id": "w_001",
        "name": "Baggy straight-leg jeans, dark wash",
        "category": "bottoms",
        "colors": [
          "dark blue",
          "indigo"
        ],
        "style_tags": [
          "denim",
          "streetwear",
          "baggy"
        ],
        "notes": "High-waisted, sits above the hip"
      },
      {
        "id": "w_002",
        "name": "Wide-leg khaki trousers",
        "category": "bottoms",
        "colors": [
          "khaki",
          "tan"
        ],
        "style_tags": [
          "earth tones",
          "minimal",
          "wide-leg"
        ],
        "notes": null
      },
      {
        "id": "w_003",
        "name": "White ribbed tank top",
        "category": "tops",
        "colors": [
          "white"
        ],
        "style_tags": [
          "basics",
          "minimal",
          "fitted"
        ],
        "notes": "Goes with everything"
      },
      {
        "id": "w_004",
        "name": "Oversized grey crewneck sweatshirt",
        "category": "tops",
        "colors": [
          "grey",
          "charcoal"
        ],
        "style_tags": [
          "oversized",
          "basics",
          "cozy"
        ],
        "notes": "Really oversized — drops below the hip"
      },
      {
        "id": "w_005",
        "name": "Black cropped zip hoodie",
        "category": "tops",
        "colors": [
          "black"
        ],
        "style_tags": [
          "athletic",
          "streetwear",
          "cropped"
        ],
        "notes": null
      },
      {
        "id": "w_006",
        "name": "Vintage black denim jacket",
        "category": "outerwear",
        "colors": [
          "black"
        ],
        "style_tags": [
          "denim",
          "vintage",
          "classic"
        ],
        "notes": "Slightly cropped"
      },
      {
        "id": "w_007",
        "name": "Chunky white sneakers",
        "category": "shoes",
        "colors": [
          "white"
        ],
        "style_tags": [
          "sneakers",
          "chunky",
          "streetwear"
        ],
        "notes": null
      },
      {
        "id": "w_008",
        "name": "Black combat boots",
        "category": "shoes",
        "colors": [
          "black"
        ],
        "style_tags": [
          "boots",
          "grunge",
          "classic"
        ],
        "notes": "Lace-up, mid-ankle height"
      },
      {
        "id": "w_009",
        "name": "Brown leather belt",
        "category": "accessories",
        "colors": [
          "brown"
        ],
        "style_tags": [
          "classic",
          "earth tones",
          "accessories"
        ],
        "notes": null
      },
      {
        "id": "w_010",
        "name": "Black crossbody bag",
        "category": "accessories",
        "colors": [
          "black"
        ],
        "style_tags": [
          "minimal",
          "accessories",
          "everyday"
        ],
        "notes": null
      }
    ]
  },
  "outfit_suggestion": "Here are two outfit suggestions using the Y2K Baby Tee with Butterfly Print and pieces from your wardrobe:\n\n**Outfit 1: Classic Y2K Streetwear**\n* New item: Y2K Baby Tee — Butterfly Print\n* Wardrobe pieces used: Baggy straight-leg jeans (w_001), vintage black denim jacket (w_006), chunky white sneakers (w_007), black crossbody bag (w_010)\n\n*Why it works:* This combination plays on the classic Y2K proportion contrast—a fitted, cropped baby tee paired with high-waisted, baggy straight-leg jeans. Adding the slightly cropped black denim jacket and chunky white sneakers leans fully into the vintage streetwear aesthetic while keeping the look balanced and cohesive.\n\n**Outfit 2: Casual Edge**\n* New item: Y2K Baby Tee — Butterfly Print\n* Wardrobe pieces used: Wide-leg khaki trousers (w_002), black cropped zip hoodie (w_005), black combat boots (w_008), brown leather belt (w_009), black crossbody bag (w_010)\n\n*Why it works:* Pairing the sweet, pink-and-purple butterfly tee with wide-leg khaki trousers creates a cool mix of earthy tones and playful Y2K graphics. Layering the black cropped zip hoodie over top (or wearing it unzipped) alongside black combat boots grounds the lighter pastel tones of the baby tee with a bit of grunge edge.",
  "fit_card": "I just scored this adorable Y2K Baby Tee with Butterfly Print on Depop for only $18, and it's giving major nostalgic vibes! I'm planning to style it with baggy straight-leg jeans and a vintage denim jacket for the ultimate classic Y2K streetwear look.",
  "error": null
}
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   Here are two outfit suggestions using the Y2K Baby Tee with Butterfly Print and pieces from your wardrobe:

**Outfit 1: Classic Y2K Streetwear**
* New item: Y2K Baby Tee — Butterfly Print
* Wardrobe pieces used: Baggy straight-leg jeans (w_001), vintage black denim jacket (w_006), chunky white sneakers (w_007), black crossbody bag (w_010)

*Why it works:* This combination plays on the classic Y2K proportion contrast—a fitted, cropped baby tee paired with high-waisted, baggy straight-leg jeans. Adding the slightly cropped black denim jacket and chunky white sneakers leans fully into the vintage streetwear aesthetic while keeping the look balanced and cohesive.

**Outfit 2: Casual Edge**
* New item: Y2K Baby Tee — Butterfly Print
* Wardrobe pieces used: Wide-leg khaki trousers (w_002), black cropped zip hoodie (w_005), black combat boots (w_008), brown leather belt (w_009), black crossbody bag (w_010)

*Why it works:* Pairing the sweet, pink-and-purple butterfly tee with wide-leg khaki trousers creates a cool mix of earthy tones and playful Y2K graphics. Layering the black cropped zip hoodie over top (or wearing it unzipped) alongside black combat boots grounds the lighter pastel tones of the baby tee with a bit of grunge edge.
  fit card: I just scored this adorable Y2K Baby Tee with Butterfly Print on Depop for only $18, and it's giving major nostalgic vibes! I'm planning to style it with baggy straight-leg jeans and a vintage denim jacket for the ultimate classic Y2K streetwear look.

=== A query it can't ===
[6] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[7] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[8] branch
      →    search returned []: stopping before suggest_outfit
=== Complete session ===
{
  "query": "designer ballgown size XXS under $5",
  "parsed": {
    "description": "designer ballgown",
    "size": "XXS",
    "max_price": 5.0
  },
  "search_results": [],
  "selected_item": null,
  "wardrobe": {
    "items": [
      {
        "id": "w_001",
        "name": "Baggy straight-leg jeans, dark wash",
        "category": "bottoms",
        "colors": [
          "dark blue",
          "indigo"
        ],
        "style_tags": [
          "denim",
          "streetwear",
          "baggy"
        ],
        "notes": "High-waisted, sits above the hip"
      },
      {
        "id": "w_002",
        "name": "Wide-leg khaki trousers",
        "category": "bottoms",
        "colors": [
          "khaki",
          "tan"
        ],
        "style_tags": [
          "earth tones",
          "minimal",
          "wide-leg"
        ],
        "notes": null
      },
      {
        "id": "w_003",
        "name": "White ribbed tank top",
        "category": "tops",
        "colors": [
          "white"
        ],
        "style_tags": [
          "basics",
          "minimal",
          "fitted"
        ],
        "notes": "Goes with everything"
      },
      {
        "id": "w_004",
        "name": "Oversized grey crewneck sweatshirt",
        "category": "tops",
        "colors": [
          "grey",
          "charcoal"
        ],
        "style_tags": [
          "oversized",
          "basics",
          "cozy"
        ],
        "notes": "Really oversized — drops below the hip"
      },
      {
        "id": "w_005",
        "name": "Black cropped zip hoodie",
        "category": "tops",
        "colors": [
          "black"
        ],
        "style_tags": [
          "athletic",
          "streetwear",
          "cropped"
        ],
        "notes": null
      },
      {
        "id": "w_006",
        "name": "Vintage black denim jacket",
        "category": "outerwear",
        "colors": [
          "black"
        ],
        "style_tags": [
          "denim",
          "vintage",
          "classic"
        ],
        "notes": "Slightly cropped"
      },
      {
        "id": "w_007",
        "name": "Chunky white sneakers",
        "category": "shoes",
        "colors": [
          "white"
        ],
        "style_tags": [
          "sneakers",
          "chunky",
          "streetwear"
        ],
        "notes": null
      },
      {
        "id": "w_008",
        "name": "Black combat boots",
        "category": "shoes",
        "colors": [
          "black"
        ],
        "style_tags": [
          "boots",
          "grunge",
          "classic"
        ],
        "notes": "Lace-up, mid-ankle height"
      },
      {
        "id": "w_009",
        "name": "Brown leather belt",
        "category": "accessories",
        "colors": [
          "brown"
        ],
        "style_tags": [
          "classic",
          "earth tones",
          "accessories"
        ],
        "notes": null
      },
      {
        "id": "w_010",
        "name": "Black crossbody bag",
        "category": "accessories",
        "colors": [
          "black"
        ],
        "style_tags": [
          "minimal",
          "accessories",
          "everyday"
        ],
        "notes": null
      }
    ]
  },
  "outfit_suggestion": null,
  "fit_card": null,
  "error": "Nothing in the listings matched description 'designer ballgown', size XXS, under $5.\nThings to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5."
}
  stopped: Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
  fit_card is None — it should still be None here

The second one should stop before the fit card. If both paths look the same,
the branch isn't doing anything yet.
(.venv) PS C:\Users\fenne\ai201-project2-fitfindr-starter-v2026> python agent.py
=== A query the data can match ===
[1] parse_query
      in:  looking for a vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: **Outfit 1: Y2K Streetwear** * New Item: Y2K Baby Tee — Butterfly Print * Wardrobe Pieces: Baggy straight-leg …
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I'm styling it with…
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   **Outfit 1: Y2K Streetwear**
* New Item: Y2K Baby Tee — Butterfly Print
* Wardrobe Pieces: Baggy straight-leg jeans, dark wash (w_001), Vintage black denim jacket (w_006), Chunky white sneakers (w_007), Black crossbody bag (w_010)

*Why it works:* This outfit leans fully into the Y2K aesthetic. The fitted crop of the baby tee balances the volume of the high-waisted, baggy dark-wash jeans. Layering the slightly cropped black denim jacket adds a cool vintage edge while keeping the silhouette balanced, and the chunky sneakers and crossbody bag tie the retro streetwear vibe together.

**Outfit 2: Casual Contrast**
* New Item: Y2K Baby Tee — Butterfly Print
* Wardrobe Pieces: Wide-leg khaki trousers (w_002), Black combat boots (w_008), Brown leather belt (w_009)

*Why it works:* This look creates a fun contrast by mixing the feminine, pastel Y2K baby tee with earth-toned, minimalist trousers. Tucking the tee in and defining the waist with the brown belt anchors the outfit, while the black combat boots add an unexpected grunge twist that grounds the lighter top and tan bottoms.
  fit card: Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I'm styling it with baggy dark-wash jeans, a black denim jacket, and chunky sneakers for the ultimate retro streetwear vibe.

=== A query it can't ===
[6] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[7] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[8] branch
      →    search returned []: stopping before suggest_outfit
  stopped: Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
  fit_card is None — it should still be None here

The second one should stop before the fit card. If both paths look the same,
the branch isn't doing anything yet.

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

- *What I asked for:* I asked ChatGPT how to make the search results easier to read because printing the list put all the listings together.
- *What came back:* It suggested printing listings separately or using json.dumps() with indentation to display each field on its own line.
- *What I changed:* I changed my per-tool terminal test to display formatted JSON. I kept the search tool returning a list of dictionaries so the agent could still use the results.

**Moment 2**

- *What I asked for:* I shared my agent code and asked where to make changes for the milestone about session state.
- *What came back:* ChatGPT identified places where the agent used local variables instead of reading stored session values. It also suggested temporarily printing the item received by the outfit tool.
- *What I changed:* I updated the search, branch, and selection steps to read from the session. I added temporary prints and compared the received item with the selected item; both contained the same complete listing. After checking, I removed the temporary item prints and kept the complete-session display.

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
| 1. matching query completes | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. impossible query stops early | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. selected item reaches outfit tool unchanged | 5 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED (0/5) |
| 4. fit cards preserve item details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. search respects different price ceilings | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
CRITERION 1 — All three tools complete — Try 1
Source: agent.py::run_agent
Tools: tools.py::search_listings, tools.py::suggest_outfit,
       tools.py::create_fit_card

[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Outfit 1: Y2K Streetwear Contrast - New Item: Y2K Baby Tee — Butterfly Print - Wardrobe Pieces: Baggy straight…
      →    10 wardrobe item(s)
[6] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I'm styling it with…

Fit card:
Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I'm styling it with baggy dark-wash jeans, a black denim jacket, and chunky sneakers for the ultimate early-2000s streetwear vibe.


CRITERION 2 — Empty search stops the agent — Try 1
Source: agent.py::run_agent and agent.py::_nothing_found_message
Report: run_eval.py::write_report

- stopped early: yes — Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
- selected_item: (none)
- search_results: 0

[3] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[4] branch
      →    search returned []: stopping before suggest_outfit


CRITERION 3 — Selected item reaches outfit tool unchanged — Try 1
Source: agent.py::run_agent

[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Outfit 1: Y2K Streetwear Contrast - New Item: Y2K Baby Tee — Butterfly Print - Wardrobe Pieces: Baggy straight…
      →    10 wardrobe item(s)

Scoring note: Marked FAIL because the report did not capture the complete
item received inside the outfit tool. Matching summaries do not verify
that all details remained unchanged. No mismatch was demonstrated.


CRITERION 4 — Fit card preserves item details — Try 1
Source: tools.py::create_fit_card
Report: run_eval.py::write_report

- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)

Fit card:
Obsessed with this Y2K Baby Tee with the cutest butterfly print, and it's up on my Depop right now for just $18. I love styling it with baggy jeans and chunky sneakers for an effortless, authentic early 2000s streetwear vibe.


CRITERION 5 — Returned listing respects the price ceiling — Try 1
Source: agent.py::parse_query and agent.py::run_agent
Search implementation: tools.py::search_listings, called through MCP

[1] parse_query
      in:  graphic tee under $18
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 2 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black
      →    2 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
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
| 1 | Matching query completes all three tools and returns a fit card | 4 of 5 | MET (5/5) | All five traces show search, outfit suggestion, and fit-card creation completing, with a non-empty caption returned. |
| 2 | Empty search stops before the outfit tool and names what to change | 5 of 5 | MET (5/5) | All five runs returned no listings, stopped before the outfit tool, and suggested changes to the search words, size, or budget. |
| 3 | First search result reaches the outfit tool with all details unchanged | 5 of 5 | MISSED (0/5 verified) | The traces show matching item summaries, but none recorded the complete argument received inside the outfit tool. I counted these as failures to demonstrate the requirement. |
| 4 | Five different listings produce captions with 2–4 sentences and the correct item, price, and platform once each | 4 of 5 | MET (5/5) | The five selected listings differ. Each caption contains 2–3 sentences and mentions the correct item, price, and platform once, accepting clear shortened item names and equivalent price formats. |
| 5 | Five searches with different price ceilings return a listing within their respective budgets | 5 of 5 | MET (5/5) | Each selected listing costs $18, within ceilings of $18, $19, $24, $27, and $30. The first try also returns an item priced exactly at the ceiling. |

**Diagnoses**
*Criterion 3 — Missing evidence at the session-to-tool handoff.*
The affected step is passing the selected item to the outfit suggestion tool. The trace summarizes the item using its title, price, and platform, but does not record the complete dictionary received inside the tool. This same evidence gap occurs in all five tries. It prevents verification of unchanged fields; it does not demonstrate that the agent passed the wrong item. The improvement is to capture a copy of the actual received argument and compare it with the first search result and selected item.

*Pattern across the misses.*
All five criterion 3 failures come from the same recording limitation, rather than five separate tool failures. Improving the evaluator’s evidence would make this requirement checkable without changing the item-selection behavior.

*Limitations of the criteria that passed.*
Criterion 5 is weaker than the intended because it requires a returned listing within budget rather than every returned listing. The current report verifies the selected item’s price, but doesn't show every result’s price. A stronger test would record all returned prices and require every one to be within the budget.

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
python app.py ask 'vintage graphic tee under $30' --trace
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Outfit 1: Y2K Streetwear Contrast - New item: Y2K Baby Tee — Butterfly Print - Wardrobe pieces: Baggy straight…
      →    10 wardrobe item(s)
[6] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Just scored this adorable Y2K Butterfly Print Baby Tee on Depop for only $18, and I am obsessed! I'm styling i…

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Outfit 1: Y2K Streetwear Contrast
- New item: Y2K Baby Tee — Butterfly Print
- Wardrobe pieces: Baggy straight-leg jeans (w_001), Vintage black denim jacket (w_006), Chunky white sneakers (w_007), Black crossbody bag (w_010)

Why it works: The fitted crop of the butterfly baby tee balances the volume of the high-waisted baggy jeans, nailing the classic 2000s silhouette. Layering the slightly cropped black denim jacket adds edge while keeping the proportions balanced, and the chunky sneakers and crossbody bag tie the streetwear theme together effortlessly.


Outfit 2: Casual Grunge Mix
- New item: Y2K Baby Tee — Butterfly Print
- Wardrobe pieces: Wide-leg khaki trousers (w_002), Black cropped zip hoodie (w_005), Black combat boots (w_008), Brown leather belt (w_009)

Why it works: This outfit plays on a sweet-meets-edgy juxtaposition. The feminine, pastel butterfly print pops against the earthy khaki trousers and tough black combat boots. Wearing the black cropped zip hoodie either open or layered over the baby tee adds an effortless, grunge-leaning Y2K vibe, accented subtly by the brown belt.

  Fit card: Just scored this adorable Y2K Butterfly Print Baby Tee on Depop for only $18, and I am obsessed! I'm styling it with baggy straight-leg jeans, a vintage black denim jacket, and chunky sneakers for the ultimate 2000s streetwear contrast.

2 model calls this session, 1311 prompt + 337 output tokens
```

**Empty search**

```
python app.py ask 'nonexistentitem' --trace
[1] parse_query
      in:  nonexistentitem
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[4] branch
      →    search returned []: stopping before suggest_outfit

  Nothing in the listings matched description 'nonexistentitem'.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'.

0 model calls this session
```

**Empty Wardrobe**

```
 python app.py ask 'graphic tee under $30' --empty-wardrobe --trace
(running with an empty wardrobe)
[1] parse_query
      in:  graphic tee under $30
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 6 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Mesh Long-Sleeve Top — Black … +3 more
      →    6 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Pair this Y2K butterfly baby tee with a low-rise pleated denim miniskirt or baggy cargo pants to lean into the…
      →    0 wardrobe item(s)
[6] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Obsessed with this Y2K Baby Tee — Butterfly Print, and it's up on my Depop right now for just $18.00! I'm pict…

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Pair this Y2K butterfly baby tee with a low-rise pleated denim miniskirt or baggy cargo pants to lean into the early 2000s aesthetic. Complement the pink and purple tones in the graphic with pastel accessories, a shoulder bag, and platform sandals or retro sneakers.

  Fit card: Obsessed with this Y2K Baby Tee — Butterfly Print, and it's up on my Depop right now for just $18.00! I'm picturing it styled with a low-rise pleated denim miniskirt or baggy cargo pants for the ultimate early 2000s streetwear vibe. Add some pastel accessories and platform sandals to really play up those pink and purple tones!

2 model calls this session, 503 prompt + 143 output tokens
```

**Model Unavailable**

```
python app.py ask 'butterfly baby tee under $29' --trace
[1] parse_query
      in:  butterfly baby tee under $29
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 5 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Low-Rise Cargo Pants — Khaki … +2 more
      →    5 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] model unavailable
      →    stopping, search results kept

  The model couldn't be reached, so the outfit and caption steps didn't run. The search worked — 5 listing(s) were found. Check GEMINI_API_KEY in your .env, then run the same query again.
What the service said: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.

1 model calls this session
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
| 1. matching query completes | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. impossible query stops early | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| empty wardrobe _(diagnostic — not one of your five)_ |  |   |   |   |   |   |  |
| 3. selected item reaches outfit tool unchanged | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. fit cards preserve item details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. search respects different price ceilings | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:
CRITERION 1 — All three tools complete — Try 1
Source: agent.py::run_agent
Tools: tools.py::search_listings, tools.py::suggest_outfit,
       tools.py::create_fit_card

[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[5] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Outfit 1: Y2K Streetwear Contrast - New Item: Y2K Baby Tee — Butterfly Print - Wardrobe Pieces: Baggy straight…
      →    10 wardrobe item(s)
[6] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Found this absolute gem of a Y2K Baby Tee with a butterfly print on Depop for just $18, and I am obsessed! To …

Fit card:
Found this absolute gem of a Y2K Baby Tee with a butterfly print on Depop for just $18, and I am obsessed! To nail that classic 2000s streetwear vibe, I’m styling it with baggy dark-wash jeans, a vintage denim jacket, and chunky sneakers to balance the fitted crop. It's giving major retro-casual energy and I can't wait to wear it out.


CRITERION 2 — Empty search stops the agent — Try 1
Source: agent.py::run_agent and agent.py::_nothing_found_message
Report: run_eval.py::write_report

- stopped early: yes — Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
- selected_item: (none)
- search_results: 0

[3] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[4] branch
      →    search returned []: stopping before suggest_outfit


CRITERION 3 — Selected item reaches outfit tool unchanged — Try 1
Agent: agent.py::run_agent
Recorded by: run_eval.py::run_once
Compared by: run_eval.py::write_report

{
  "first_search_result": {
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
  "selected_item": {
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
  "items_received_by_outfit_tool": [
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
    }
  ],
  **"all_details_match": true**
}


CRITERION 4 — Fit card preserves item details — Try 1
Source: tools.py::create_fit_card
Report: run_eval.py::write_report

- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)

Fit card:
Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed with the pink and purple print! I'm planning to style it for a nostalgic Y2K streetwear vibe by pairing it with baggy straight-leg jeans, a vintage black denim jacket, and chunky white sneakers to balance the fitted crop.


CRITERION 5 — Returned listing respects the price ceiling — Try 1
Source: agent.py::parse_query and agent.py::run_agent
Search implementation: tools.py::search_listings, called through MCP

[1] parse_query
      in:  graphic tee under $18
      out: dict with keys: description, size, max_price
[2] MCP search succeeded
      →    Search results received from the MCP tool.
[3] search_listings
      in:  dict with keys: description, size, max_price
      out: 2 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black
      →    2 match(es)
[4] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)

**Did it help, and how do I know:**
The improvement made criterion 3 verifiable: the before report had 0/5 verified comparisons, while the after report recorded matching complete dictionaries in 5/5 tries. This improved the evaluation evidence without changing the agent’s item-selection behavior. The other four criteria remained MET.
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
