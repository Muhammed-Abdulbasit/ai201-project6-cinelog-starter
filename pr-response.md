# PR Response Doc — CineLog Watchlist Feature
Comment 1 — Rename: Rename save_to_watchlist() to add_to_watchlist() in services/watchlist_service.py and update all call sites (there is one in routes/watchlist/watchlist.py). Use your editor's find-all-references or a project-wide search to confirm you haven't missed any. Commit this change.


Comment 2 — Deduplication: Add deduplication logic to add_to_watchlist() in services/watchlist_service.py. Look at how add_to_collection() in services/collection_service.py handles this — follow the same pattern. Commit this change separately from the rename.


Comment 3 — Missing test: Create a new file tests/test_watchlist.py. Read tests/test_collection.py and find test_add_to_collection_nonexistent_film_raises — write the equivalent test for add_to_watchlist() following the same fixture and assertion structure. Run the test to confirm it passes:

pytest tests/test_watchlist.py -v
pytest tests/ -v

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the one call site in `routes/watchlist/watchlist.py` (see commit `fe54a72`).
**How I verified:** Searched the project for any remaining references to `save_to_watchlist` and confirmed none were left, then re-ran the existing test suite to confirm nothing else called the old name.

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyInWatchlistError` exception and a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, mirroring the pattern in `add_to_collection()` (`services/collection_service.py`): query for an existing `WatchlistEntry` with the same `user_id`/`film_id` before inserting, and raise if one already exists. Also updated `routes/watchlist/watchlist.py` to catch `AlreadyInWatchlistError` and return a 409, matching how `routes/collection.py` handles `AlreadyInCollectionError`.
**How I verified:** Manually exercised `add_to_watchlist()` against an in-memory SQLite app context: adding a film the first time succeeds, adding the same film again raises `AlreadyInWatchlistError`, and adding a nonexistent `film_id` still raises `FilmNotFoundError`. Also re-ran the existing `tests/test_collection.py` suite (4 passed) to confirm the shared `FilmNotFoundError` import wasn't broken.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` mirroring `tests/test_collection.py`'s fixtures and assertion structure. `test_add_to_watchlist_nonexistent_film_raises` is the direct equivalent of `test_add_to_collection_nonexistent_film_raises` as requested. I also added `test_add_to_watchlist_creates_entry`, `test_add_to_watchlist_duplicate_raises`, and `test_get_watchlist_returns_films_sorted_by_title` so the new service function has the same three categories of coverage (happy path, duplicate/conflict, nonexistent ID) CONTRIBUTING.md requires for new service functions, plus a sort-order check for `get_watchlist()`. Writing that last test surfaced a real bug: `Film` had a `collection_entries` relationship backing `entry.film` for `CollectionEntry`, but no equivalent relationship for `WatchlistEntry`, so `get_watchlist()` crashed with `AttributeError: 'WatchlistEntry' object has no attribute 'film'`. Fixed by adding `watchlist_entries = db.relationship("WatchlistEntry", backref="film", lazy=True)` to `Film` in `models.py`, mirroring the existing `collection_entries` relationship.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` and `pytest tests/ -v` as instructed — all 8 tests pass (4 existing collection tests + 4 new watchlist tests). Before the `models.py` fix, `test_get_watchlist_returns_films_sorted_by_title` failed with the `AttributeError` above, confirming the test actually catches the bug rather than passing vacuously.

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
