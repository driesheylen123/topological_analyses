# Intro
In the BioMOBS workflow this tool for topological analyses integrates with MOBS (https://mobs.vercel.app/) and CKG (https://ckg.readthedocs.io/en/latest/INTRO.html) to perform clinically relevant pathway analyses. This repository is based on a copy/paste/modify approach for creating visualisation dashboards. Users can format their data here https://driesheylen.shinyapps.io/shiny/. Users should store their Formatted data in the 'src/routes/api' folder of this repository.

## Running the app locally
This project is developped in the JavaScript framework [Svelte](https://svelte.dev). To run this application locally, clone the repository to a directory on your computer. *Note that you will need to have [Node.js](https://nodejs.org) installed.*

1. Open the terminal/command promt on your pc.
2. change the directory to the folder you want to store the application in.
```bash
cd Documents\Biomobs_topological_analyses
```
3. Clone this GitHub repository into your folder.
```bash
git clone https://github.com/driesheylen123/BioMOBS-topological-analyses.git
```

5. Install all necessary dependencies. *Make sure [Node.js](https://nodejs.org) is installed on your device*
```bash
npm install
```
6. Run the application
```bash
npm run dev
```
7. Navigate to [localhost:3000](http://localhost:3000). You should now see the app running.



## Assumptions
- The data is read from an ArangoDB database.
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
- Any reshaping of the data (e.g. adding a `source` and `target` to edges if only `_from` and `_to` were defined) is done in the endpoints. Example formats of how  the data must be stored in the ArangoDB database is demonstrated in the 'data/' folder. The json files there can be directly stored in a ArangoDB database and called in the endpoints for a try-out of this design. The 'data/BioMOBS_stad.ipynb' file shows how you can read the data to apply this toplogicial analyses visual.

## Inspiration:
- https://gitlab.com/JelmerBot
- https://sandeep.ramgolam.com/blog/degit
- See https://www.sitepoint.com/a-beginners-guide-to-sveltekit/
- https://www.youtube.com/watch?v=a5OiuEu1Q6M
