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
**My position:** Keep public=True as the default.

**Reasoning:** Most people using CineLog are watching movies from their personal accounts and want to share what they're watching with friends. Making watchlists public by default supports that — it's a natural way to discover recommendations from people you know without adding friction.

**Tradeoff acknowledged:** Privacy is a real concern. Some users may not want their watchlist public. We should add a setting later so users can make their watchlist private if they prefer, but for now, defaulting public makes sense for the social sharing use case.

## Comment 5 — Sort order
**My position:** Sort by date added (newest first).

**Reasoning:** When I use a watchlist, I want to see what I added recently because those movies are still fresh in my mind. But I also care about seeing what I added at different points — like going back to films I added at the beginning vs. recently. Alphabetical order doesn't help with either of those — it just alphabetizes titles, which isn't useful for deciding what to watch.

**Engagement with reviewer's point:** I agree with the reviewer that recency matters. People want to see their recent additions first, not just an alphabetical list.

## Comment 6 — Rebase
**What conflicted:** The main branch refactored film IDs from integers to UUIDs. The WatchlistEntry model and all references to film_id needed to be updated to use db.String(36) instead of db.Integer.

**How I resolved it:** Updated the WatchlistEntry model in models.py to use db.String(36) for film_id, matching the Film model's UUID type. Updated the docstring in watchlist_service.py to reflect that film_id is now a UUID string. Updated the test file to use a fake UUID instead of an integer.

**How I verified no conflict remains:** Ran `pytest tests/ -v` — all 7 tests pass. Checked git log and confirmed no merge commits remain in the branch history.

## PR Description

### Feature Overview
Added a complete watchlist feature for CineLog. Users can now:
- Add films to their watchlist via the POST /watchlist/<user_id>/add endpoint
- View their watchlist via the GET /watchlist/<user_id> endpoint
- Remove films from their watchlist (future stretch feature)

The feature includes deduplication (users can't add the same film twice), proper error handling for nonexistent films, and full test coverage.

### Design Decisions

1. **Default Visibility: Public** — Watchlist entries are public by default to encourage sharing recommendations with friends. This supports the community aspect of CineLog without adding friction. Users can make their watchlist private in a future update if needed.

2. **Sort Order: Date Added (Newest First)** — Watchlists are sorted by date_added in descending order so users see their most recent additions first. This matches how users naturally interact with watchlists — they want to see what they just added because it's still fresh in their mind.

### Manual Testing Steps

1. Start the app: `python app.py`

2. Create a test user and film (or use existing UUIDs from the database):
   ```
   # Get a user UUID and film UUID from your database first
   ```

3. Add a film to watchlist:
   ```
   curl -X POST http://127.0.0.1:5000/watchlist/USER_UUID/add \
     -H "Content-Type: application/json" \
     -d '{"film_id": "FILM_UUID"}'
   ```
   Expected: Returns 201 status and the watchlist entry object.

4. View the watchlist (sorted by date added, newest first):
   ```
   curl http://127.0.0.1:5000/watchlist/USER_UUID
   ```
   Expected: Returns all films on the watchlist in chronological order (newest first).

5. Try adding a duplicate film:
   ```
   curl -X POST http://127.0.0.1:5000/watchlist/USER_UUID/add \
     -H "Content-Type: application/json" \
     -d '{"film_id": "FILM_UUID"}'
   ```
   Expected: Returns error indicating film is already in watchlist.

6. Try adding a nonexistent film:
   ```
   curl -X POST http://127.0.0.1:5000/watchlist/USER_UUID/add \
     -H "Content-Type: application/json" \
     -d '{"film_id": "00000000-0000-0000-0000-000000000000"}'
   ```
   Expected: Returns error indicating film not found.
