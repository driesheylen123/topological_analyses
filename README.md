# Intro
In the BioMOBS workflow this tool for topological analyses integrates with MOBS (https://mobs.vercel.app/) and CKG (https://ckg.readthedocs.io/en/latest/INTRO.html) to perform clinically relevant pathway analyses. This repository is based on a copy/paste/modify approach for creating visualisation dashboards. 

## Usage
- `npx degit https://gitlab.com/vda-lab/viskit --mode=git new_project`
- `cd new_project`
- `npm i`
- Create `.env` (see below)
- `npm run dev`

## Assumptions
- The data is from an ArangoDB database.
- Objects in document collections have an `id` attribute which is a copy of the `_id`.
- Objects in edge collections have a `source` attribute which is a copy of `_from`, and a `target` which is a copy of `_to`.

## Approach
- The database is defined in `src/lib/database.js`.
- The connection details (host, database name, username, password) are stored in a `.env` file at the root level. This file is _not_ put under source code control obviously, so you'll have to create this file every time you make a clone of this repo. The format of the file looks like this:

```
VITE_DB_URL=your_database_url
VITE_DB=your_database_name
VITE_USER=your_user
VITE_PASSWORD=your_password
```

- The actual access through the data is defined in endpoints, defined in `routes/api` (OBSOLETE files). Typically, there are different endpoints for different datasets or types of data. For example, pointing to different tables in the database if this makes sense. These endpoints basically contain the AQL query to get the data from the database. TIP: while developing, check the contents of these endpoints by going to e.g. http://localhost:3000/api/datapoints.json.
- Any reshaping of the data (e.g. adding a `source` and `target` to edges if only `_from` and `_to` were defined) is done in the endpoints. Example formats of how  the data must be stored in the ArangoDB database is demonstrated in the 'data/' folder. The json files there can be directly stored in a ArangoDB database and called in the endpoints for a try-out of this design. The 'data/BioMOBS_stad.ipynb' file shows how you can process your data to apply this toplogicial analyses visual.

## Inspiration:
- https://gitlab.com/JelmerBot
- https://sandeep.ramgolam.com/blog/degit
- See https://www.sitepoint.com/a-beginners-guide-to-sveltekit/
- https://www.youtube.com/watch?v=a5OiuEu1Q6M
