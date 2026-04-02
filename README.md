# Dimensional Data Modeling - Actor Films Dataset

Picture a vibrant film industry where actors' careers evolve, and we need to track their performances over decades. The `actor_films` dataset is our raw footage, packed with details about actors and their movies. This repository transforms it into a structured, analytical masterpiece using SQL, creating tables to analyze actors' current profiles and historical changes.

<img width="2752" height="1536" alt="image" src="https://github.com/user-attachments/assets/6ddc6f5f-e76e-4d78-be0f-c5140c4703b2" />

## Project Overview

- The `actor_films` dataset captures actors, films, and metrics like ratings and votes (from 1970 - 2021).
- We build a dimensional model with two tables: `actors` for yearly snapshots and `actors_history_scd` for tracking changes over time using Slowly Changing Dimension (SCD) Type 2.

### Dataset Description

- **Fields**:
  - `actor`: Actor's name.
  - `actorid`: Unique actor ID.
  - `film`: Film name.
  - `year`: Release year.
  - `votes`: Number of votes.
  - `rating`: Film rating.
  - `filmid`: Unique film ID.
- **Primary Key**: (`actorid`, `filmid`).
  
<img width="1006" height="636" alt="Screenshot 2025-09-03 at 10 56 31" src="https://github.com/user-attachments/assets/9f19f4a7-2b18-43c2-97b0-f66e670bd471" />

## Key Concepts
- **Why create actors and actors_history_scd?**:
  - The raw dataset actor_films is transactional – it shows films year by year.
  - We need a dimension table (actors) that summarizes each actor, their films, and yearly performance.
  - We also need a history table (actors_history_scd) to keep track of how an actor’s status (quality_class, is_active) changes over time.


- **SCD Type 2**: keeping a full history of changes
  - Tracks historical changes (e.g., `quality_class`, `is_active`) by adding new rows per change.
  - Uses `start_date` and `end_date` to mark record validity.
  - Preserves history without overwriting.
  - Instead of overwriting old values, we keep multiple records with start_date and end_date.
Example: If an actor moves from good → star, we close the old record and start a new one.

- **Backfill**: Filling the entire history at once from scratch
  - Loads all historical data into a table at once (e.g., `actors_history_scd` up to 2020).
  - Like filming an entire movie in one go, capturing the full backstory.
  - Scan all past data and generate complete SCD records (like recreating history from 1970–2021).


- **Incremental**: Updating the SCD year by year with only new changes.
  - Adds only new or changed data (e.g., 2021 data).
  - Like shooting a new scene to continue the story.
Example: At the end of 2021, compare with 2020, and only insert records for actors whose status changed or new actors appeared.


- **When to Use Backfill vs. Incremental**:
  - **Backfill**:
    - **Use When**: Initializing a new table, catching up on historical data, or rebuilding after data loss.
    - **Pros**:
      - Comprehensive: Captures all historical data.
      - Ensures consistency across the dataset.
      - Ideal for one-time setups or corrections.
    - **Cons**:
      - Resource-intensive: Processes large data volumes.
      - Slow for big datasets.
      - Risk of redundant processing if data already exists.
  - **Incremental**:
    - **Use When**: Adding new data periodically (e.g., daily, yearly updates).
    - **Pros**:
      - Efficient: Processes only new/changed data.
      - Faster for ongoing updates.
      - Scalable for frequent updates.
    - **Cons**:
      - Requires existing historical data.
      - Risk of missing data if prior steps fail.
      - Complex to manage dependencies.

- **Why Create `actors` and `actors_history_scd`?**:
  - `actor_films`: Raw, transactional data, not optimized for analysis.
  - `actors`: Aggregates film data per actor per year for quick insights.
  - `actors_history_scd`: Tracks quality and activity changes over time for trend analysis.
    
## Assignment Tasks

1. **DDL for `actors` Table**:
   - Stores actor info, film arrays, quality class (star/good/average/bad), and active status.
   - Uses custom types `film_stats` and `quality_check`.

2. **Cumulative Table Generation**:
   - Populates `actors` year by year (1971–2021).
   - Aggregates films, calculates quality based on ratings.

3. **DDL for `actors_history_scd` Table**:
   - SCD Type 2 to track `quality_class` and `is_active` changes.
   - Includes `start_date` and `end_date`.

4. **Backfill Query for `actors_history_scd`**:
   - Populates historical data up to 2020.
   - Uses window functions to detect changes.

5. **Incremental Query for `actors_history_scd`**:
   - Updates 2021 data, handling unchanged, changed, and new records.

## Notes

- `actors` uses `current_year` for snapshots.
- `actors_history_scd` uses 2020 for backfill, 2021 for incremental updates.
- Ensure `actor_films` spans 1971–2021.
- Incremental query assumes 2021 data in `actors`.

## Resume Highlights:

- **Built Dimensional Data Model:** Designed SQL-based `actors_history_scd` table and `actors` table for the `actor_films` dataset, enabling efficient performance analysis.  
- **Implemented SCD Type 2:** Created backfill and incremental queries to track actor quality and activity changes (1971–2021) in PostgreSQL.  

## Output
- Actors: summarizes each actor, their films, and yearly performance
  <img width="1130" height="687" alt="Screenshot 2025-09-03 at 11 33 23" src="https://github.com/user-attachments/assets/b6573f9a-524a-4d0f-a576-9cdedfc8c45e" />

- Backfill actors_history_scd (from 1971 - 2020: the year before 2021 (max year from the data)
<img width="684" height="700" alt="Screenshot 2025-09-03 at 11 40 31" src="https://github.com/user-attachments/assets/6ddec306-f9df-49f0-b4ce-8cfb902103c4" />

_ Incremental Query (Add new data from 2021 to the backfill 2020):
<img width="745" height="845" alt="Screenshot 2025-09-03 at 11 45 14" src="https://github.com/user-attachments/assets/4bc2a159-3d42-491a-85b7-699aedd15e9d" />
