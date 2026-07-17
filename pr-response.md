# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used AI to help me understand deduplication patterns and understand the codebase overall.

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
**My position:** `public` should default to `False`, not `True`. I'd change `public = db.Column(db.Boolean, default=True)` to `default=False` in `WatchlistEntry` (a follow-up code change, not made as part of this response, since the comment only asked for a written position).
**Reasoning:** A watchlist entry reveals *intent* — what a user is thinking about watching next — which is more exposing than a collection entry, which just records something already watched. People add things to a watchlist impulsively, off a recommendation, a trailer, or a search, often without pausing to consider who else can see it. `CollectionEntry` doesn't even have a visibility flag, so this is the first place in the codebase where "public by default" is a real design choice, not an inherited convention. Defaulting to private optimizes for the common case where a user is just building a personal to-watch list for themselves, and makes sharing an explicit, considered action rather than something that happens by not noticing a checkbox. That's the safer failure mode: if we default to private and a user wanted to share, the cost is they don't get social visibility until they flip it on; if we default to public and a user didn't want to share, the cost is an unintended disclosure they can't take back once someone's seen it.
**Tradeoff acknowledged:** Defaulting to private does add friction to the feature's social value — CineLog is a film-logging app where the whole point is discovery through what other people are watching, and a private-by-default watchlist means fewer entries are visible for friends to see, weakening the network effect that makes the feature worth using. A public-by-default watchlist would drive more engagement with zero effort from the user. I think that engagement cost is worth paying for a feature carrying more personal signal than the collection, but it's a real cost, not a free win.

## Comment 5 — Sort order
**My position:** I implemented the maintainer's preference — `get_watchlist()` now sorts by `date_added` descending (newest first), replacing the alphabetical-by-title order. See `services/watchlist_service.py`.
**Reasoning:** Newest-first is the only ordering in this codebase that's actually tied to user behavior instead of an incidental string property. `get_collection()` already sorts newest-first, so this makes the two list endpoints behave consistently — a user shouldn't have to learn two different sort conventions for two visually similar screens. It also fits the watchlist's actual use case better: the film you added most recently is the one freshest in your mind and most likely to be what you're deciding between right now, so surfacing it at the top is more useful than an alphabetical position that has no relationship to why the film is on the list.
**Engagement with reviewer's point:** The maintainer's argument was presumably some version of "match the collection's ordering and reflect recency of user action" rather than an arbitrary property of the film metadata. I agree with that reasoning rather than just deferring to it: alphabetical sort was my original implementation, but on reflection it optimizes for browsing an already-large list, not for the more common case of "what did I just add and might want to watch tonight." If the watchlist grows very long, alphabetical (or a dedicated search/filter) might become more useful again — but that's a scaling concern for a future iteration, not a reason to pick it as the default now. I updated `test_get_watchlist_returns_newest_first` (previously `test_get_watchlist_returns_films_sorted_by_title`) to assert the new order, mirroring `test_get_collection_returns_newest_first`'s structure.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
