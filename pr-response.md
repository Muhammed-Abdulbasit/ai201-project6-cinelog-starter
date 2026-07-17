# PR Response Doc — CineLog Watchlist Feature
Comment 1 — Rename: Rename save_to_watchlist() to add_to_watchlist() in services/watchlist_service.py and update all call sites (there is one in routes/watchlist/watchlist.py). Use your editor's find-all-references or a project-wide search to confirm you haven't missed any. Commit this change.


Comment 2 — Deduplication: Add deduplication logic to add_to_watchlist() in services/watchlist_service.py. Look at how add_to_collection() in services/collection_service.py handles this — follow the same pattern. Commit this change separately from the rename.

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the one call site in `routes/watchlist/watchlist.py` (see commit `fe54a72`).
**How I verified:** Searched the project for any remaining references to `save_to_watchlist` and confirmed none were left, then re-ran the existing test suite to confirm nothing else called the old name.

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyInWatchlistError` exception and a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, mirroring the pattern in `add_to_collection()` (`services/collection_service.py`): query for an existing `WatchlistEntry` with the same `user_id`/`film_id` before inserting, and raise if one already exists. Also updated `routes/watchlist/watchlist.py` to catch `AlreadyInWatchlistError` and return a 409, matching how `routes/collection.py` handles `AlreadyInCollectionError`.
**How I verified:** Manually exercised `add_to_watchlist()` against an in-memory SQLite app context: adding a film the first time succeeds, adding the same film again raises `AlreadyInWatchlistError`, and adding a nonexistent `film_id` still raises `FilmNotFoundError`. Also re-ran the existing `tests/test_collection.py` suite (4 passed) to confirm the shared `FilmNotFoundError` import wasn't broken.

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
