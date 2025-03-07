[![No Maintenance Intended](http://unmaintained.tech/badge.svg)](http://unmaintained.tech/)

Jeopardy parser
===============

Quick note: this project does **not** use semantic versioning (`python parser.py --version` outputs the last updated date of the script).

What is this?
-------------

This is a Python3 script to extract [Jeopardy!] clues from the [J! Archive] website and dump them into a SQLite database for use elsewhere (no particular application is intended). Python 3.x and SQLite on *nix have been tested and confirmed to work (requires BeautifulSoup 4 and the lxml parser).

  [Jeopardy!]:http://www.jeopardy.com/
  [J! Archive]:http://j-archive.com/


This version is modified to have a Links item in the database containing the links to images in clues where they exist to use with the palbot discord bot. It's also modified to take both the old and new html of the j-archive pages automatically.

Quick start
-----------

```bash
git clone https://github.com/iamsix/jeopardy-parser.git
cd jeopardy-parser
pip install -r requirements.txt
python3 download.py
python3 parser.py
```

How long will all this take?
----------------------------

There are two important steps:

1. Downloading the game files from the J! Archive website
2. Parsing and inserting them into the database

The first step, downloading, will depend on the machine: the download script will pause between each download based on the `SECONDS_BETWEEN_REQUESTS` const at the top of download.py. The second step, parsing, should take ~30 minutes (on a 1.7 GHz Core i5 w/ 4 GB RAM). In total, you're looking at around 2 hours (probably less).

The complete download of the game files is ~350MB, and the resulting database file is ~50MB (although these numbers are qucikly outdated as the number of games increases).

Querying the database
---------------------

The database is split into 5 tables:

| Table name        | What it holds                                          |
| ----------------- | ------------------------------------------------------ |
| `airdates`        | Airdates for the shows, indexed by game number         |
| `documents`       | Mappings from clue IDs to clue text and answers        |
| `categories`      | The categories                                         |
| `clues`           | Clue IDs with metadata (game number, round, and value) |
| `classifications` | Mappings from clue IDs to category IDs                 |

To get all the clues along with their metadata:

```sql
SELECT clues.id, game, round, value, clue, answer
FROM clues
JOIN documents ON clues.id = documents.id
-- WHERE <expression>
;
```

To get the category that a clue is in, given a clue id:

```sql
SELECT clue_id, category
FROM classifications
JOIN categories ON category_id = categories.id
-- WHERE <expression>
;
```

To get everything (although it is better to pick and choose what you're looking for):

```sql
SELECT clues.id, clues.game, airdate, round, value, category, clue, answer
FROM clues
JOIN airdates ON clues.game = airdates.game
JOIN documents ON clues.id = documents.id
JOIN classifications ON clues.id = classifications.clue_id
JOIN categories ON classifications.category_id = categories.id
-- WHERE <expression>
;
```

License
-------

This software is released under the MIT License. See the [LICENSE.md](LICENSE.md) file for more information.
