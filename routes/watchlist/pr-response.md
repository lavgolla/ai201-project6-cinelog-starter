# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did: Identified the save_to_watchlist() across application and replaced with add_to_watchlist()**
**How I verified: rechecked if the save_to_watchlist() still exist and also checked using grep -rn "save_to_watchlist" /Users/lavanya/ClassAi201/ai201-project6-cinelog-starter --include="*.py" it returns none**

## Comment 2 — Deduplication
**What I did:I dentify the code from logic from add_to_collection() in services/collection_service.py Add deduplication logic to add_to_watchlist() in services/watchlist_service.py**
**How I verified: Understand where the logic sits and added the deduplication logic**

## Comment 3 — Missing test
**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
