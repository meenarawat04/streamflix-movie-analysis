# StreamFlix Data Cleaning Log

All cleaning was done in Excel. Original files are kept in `data/raw/` and cleaned versions in `data/cleaned/`.

## 1. Initial Data Findings

| Table | Records | Primary Key | Foreign Keys | Missing Values | Duplicates |
|---|---|---|---|---|---|
| Movies | 3,883 | MovieID | none | No | No |
| Users | 6,040 | UserID | none | No | No |
| Ratings | 10,000 | RatingID | UserID (Users), MovieID (Movies) | No | No |

Checks on `MovieID`, `Title`, `UserID` and `RatingID` used Conditional Formatting to highlight missing and duplicate values. None were found, so the Users and Ratings tables needed no changes.

## 2. Movies Table Cleaning

The Movies table was cleaned to meet specific client requirements.

| # | Issue | Column | Method | Action taken |
|---|---|---|---|---|
| 1 | Untitled film: MovieID 3936 had no title and only the value `-1943` | Title | Sorted Title A to Z to find the record | Replaced with `The Phantom of the Opera (1943)` |
| 2 | Missing year: MovieID 16 was titled `Casino (not_def)` | Title | Find and Replace on `not_def` | Replaced with 1995, giving `Casino (1995)` |
| 3 | Year was embedded in the title | Year (new column) | Excel formula `=MID(B2,LEN(B2)-4,4)` | Created a new Year column, then pasted as values |
| 4 | Title contained the year, e.g. `Toy Story (1995)` | Title | Excel formula `=LEFT(B2,LEN(B2)-6)` | Pasted as values, removed the old column, and kept the title only |
| 5 | Inconsistent genre separator: MovieID 3875 used `--` instead of `\|` | Genres | Find and Replace | Replaced `--` with `\|` |
| 6 | Spelling error: `Dramma` instead of `Drama` | Genres | Filter and Find and Replace | Corrected to `Drama` (for example MovieID 608 and 3317) |

### Reviewed with no change
- MovieID 2031 starts with `$` and MovieID 3420 starts with `...`. These are genuine titles (`$1,000,000 Duck` and `...And Justice for All`), so they were kept as they are.
- No missing or duplicate values were found in `MovieID` or `Title` apart from the issues above.

## 3. Final Check

After cleaning, the Movies table has:
- 3,883 records with unique MovieIDs
- A separate `Year` column (1919 to 2000)
- Titles with no year in the text
- Consistent `|` separators and no `Dramma` spellings in `Genres`

## 4. Genre Transformation (Power Query and DAX)

Many films have several genres (for example `Comedy|Drama`). Counting these as separate categories would understate single-genre totals, so I created a separate table for the genre analysis.

| Step | Detail |
|---|---|
| Reference the table | Created `MoviesGenres` as a reference of Movies, keeping `MovieID`, `Genres` and `Total_Views` |
| Split into rows | Split `Genres` by the `\|` delimiter into **rows**, so each film appears once per genre |
| Clean the values | Trimmed spaces and renamed the column to `Genre` |
| Relationship | Linked `Movies[MovieID]` to `MovieGenres[MovieID]` (one-to-many) |
| Weighted views | Created a DAX measure that divides each film's views equally across its genres, so genre totals add up to total views |

Result: the Top 10 Genres chart now shows Drama (59M), Comedy (42M) and Thriller (14M) as the top three, ahead of Horror (13M), Action (12M) and Romance (12M).

## 5. Known Limitation

Weighted views assume a film's views are split equally between its genres. Counting a film's full views under every genre would give higher totals and could change the ranking slightly.
