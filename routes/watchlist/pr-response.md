# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Identified every call to `save_to_watchlist()` across the codebase and renamed it to `add_to_watchlist()` in `services/watchlist_service.py` and all call sites.

**How I verified:** Ran `grep -rn "save_to_watchlist" . --include="*.py"` — returned no output, confirming the old name is fully gone. Also confirmed `add_to_watchlist` appears in the service and route files.

## Comment 2 — Deduplication
**What I did:** Modeled the deduplication logic on `add_to_collection()` in `services/collection_service.py`. Added a check in `add_to_watchlist()` that queries for an existing `WatchlistEntry` with the same `user_id` and `film_id` before inserting, and raises an error if one is found.

**How I verified:** Traced the logic against the collection service pattern to confirm the guard runs before the insert. The existing `AlreadyInCollectionError` pattern was the reference.

## Comment 3 — Missing test
**What I did:** Wrote `test_add_to_watchlist_nonexistent_film_raises` in `tests/test_watchlist.py`. The test uses the same `app` and `sample_user` fixtures as the collection tests, passes a fake `film_id` of `999999` that does not exist in the database, and asserts that `FilmNotFoundError` is raised.

**How I verified:** Ran `pytest tests/test_watchlist.py -v` — 1 passed in 0.24s with no errors.

## Comment 4 — Default visibility
**My position:** Keep `public=True` as the default.

**Reasoning:** The watchlist is a discovery and social feature — its value grows when users can share what they plan to watch with friends or see each other's lists. Defaulting to public optimizes for that use case: a new user who adds a film and never touches the visibility setting still participates in the social layer without having to find a toggle. This matches how platforms like Letterboxd and Goodreads approach it — public by default lowers the friction for the behavior the feature is designed to encourage.

**Tradeoff acknowledged:** The real cost of `public=True` is that users who add films they don't want others to see — films they're embarrassed about, gifts they're researching, private recommendations — are exposed by default unless they remember to flip the setting. For a privacy-first product that tradeoff would be unacceptable. For CineLog, where the social dimension is central to the feature's purpose, I think it's the right call, but it does mean the UI must make the visibility setting prominent and easy to find so users who want privacy aren't surprised.

## Comment 5 — Sort order
**My position:** Implement date-added descending (most recently added first), as the reviewer suggested.

**Reasoning:** A watchlist is a queue of intent — users add films when they're motivated to watch them, and that motivation is most fresh for recent additions. Sorting by date-added puts the films the user is most likely thinking about right now at the top. Alphabetical sort, which the current code uses, treats all entries as equally important and optimizes for lookup by name — that's useful for a reference list, but a watchlist is not a reference list, it's a to-watch queue.

**Engagement with reviewer's point:** The reviewer's argument is essentially that recency signals priority, and I agree. The one case where alphabetical wins is a very large watchlist where the user knows the title they're looking for — but that's a search/filter problem, not a sort problem. The default sort should serve the common case (browsing what to watch next), not the edge case (finding a specific title). Date-added descending is the right default, and a future sort toggle can address the edge case without compromising the primary experience.

## Comment 6 — Rebase
**What conflicted:** `models.py` conflicted due to main branch commit `07ca580` ("refactor: migrate film IDs from integer to UUID"). That commit changed two things this branch also touched: `Film.id` was changed from `db.Column(db.Integer, primary_key=True, autoincrement=True)` to `db.Column(db.String(36), ...)` with UUID default, and `CollectionEntry.film_id` was changed from `db.Integer` to `db.String(36)` foreign key. This branch added `WatchlistEntry` with `film_id = db.Column(db.Integer, ...)`, which directly conflicted with the UUID schema on main.

**How I resolved it:** Kept this branch's integer types for `Film.id`, `CollectionEntry.film_id`, and `WatchlistEntry.film_id`. The watchlist feature was built and tested against the integer schema, and silently adopting UUID here without a proper database migration would break existing data. Resolving the integer→UUID migration is a separate concern to be handled after this PR merges, coordinated with the team.

**How I verified no conflict remains:** Ran `git status` — no conflict markers present in any file. Ran `pytest tests/test_watchlist.py -v` and `pytest tests/test_collection.py -v` — all tests pass. Ran `grep -r "<<<<<<" . --include="*.py"` — no output, confirming no leftover conflict markers in source files.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
