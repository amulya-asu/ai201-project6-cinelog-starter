# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**

**How I verified:**

## Comment 2 — Deduplication
**What I did:**

**How I verified:**

## Comment 3 — Missing test
**What I did:**
Created `tests/test_watchlist.py` with three test functions following the same pattern as `tests/test_collection.py`:
- `test_add_to_watchlist_creates_entry` — Verifies that adding a valid film creates a WatchlistEntry in the database
- `test_add_to_watchlist_duplicate_raises` — Verifies that adding a duplicate film raises ValueError and only one entry is created
- `test_add_to_watchlist_nonexistent_film_raises` — Verifies that adding a nonexistent film_id raises FilmNotFoundError

Used integer film_id (99999) as a fake ID since the feature branch is pre-UUID refactor.

**How I verified:**
Ran `pytest tests/test_watchlist.py -v` — all 3 tests pass. Also ran full test suite with `pytest tests/ -v` — all 7 tests pass (4 collection + 3 watchlist).

Also fixed an import error in `routes/watchlist/watchlist.py` where it was trying to import the old `save_to_watchlist` name instead of `add_to_watchlist`.

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
