# Gans — Predicting Where E-Scooters Are Needed

A data pipeline project I built during my data analytics training. The idea behind it: if a city-based e-scooter company wants to put scooters where people actually need them, what outside data could help them predict demand — and how do you collect and store it automatically?

---

## The Scenario

**Gans** is an e-scooter sharing startup operating across European cities. Scooters only make money when they're in the right place at the right time — an empty scooter downtown while a crowd lands at the airport is a missed ride.

A big part of Gans' customers are **tourists**: people who fly in with just a backpack and want a quick, convenient way to get around. So the company needs outside data that hints at *where and when* demand will spike.

My job was to build a pipeline that collects that external data, cleans it, and stores it in a database Gans could actually use.

> **The question:** What external data predicts scooter demand — and how do we collect it reliably and automatically?

---

## What I Wanted to Build

- A way to gather **city facts** (size, population, density) to spot the busiest markets
- **Weather forecasts**, since rain kills scooter demand
- **Flight arrivals**, to know when waves of tourists land
- **Local events**, because a concert or festival sends demand soaring
- A clean **database** tying it all together, ready for analysis

---

## The Data Sources

| Source | Method | What it tells Gans |
|---|---|---|
| Wikipedia | Web scraping (BeautifulSoup) | Which cities are big and dense enough to matter |
| OpenWeather | REST API | When rain will suppress demand |
| AeroDataBox (via RapidAPI) | REST API | When tourists are landing |
| Ticketmaster | REST API | When events will spike local demand |

---

## How I Approached It

I built the pipeline one data source at a time, and every source followed the same four moves: **request → parse → extract → store**. Keeping that pattern consistent made each new source easier than the last.

### 1. City Facts (Web Scraping)

There's no clean API for "give me facts about this city," so I scraped each city's Wikipedia page with BeautifulSoup, pulling population, area, elevation, and coordinates from the infobox. I wrote small reusable helper functions to handle the messy parts — footnotes, commas inside numbers, and the fact that capitals label their country differently from other cities.

### 2. Weather Forecasts (OpenWeather API)

My first real API. I pulled a 5-day forecast for every city — temperature, rain probability, wind — and learned that weather is a *time series*: many rows per city, one per timestamp, not duplicates.

### 3. Flight Arrivals (AeroDataBox via RapidAPI)

The trickiest source. I collected tomorrow's arrivals for each city's airport, and this one taught me the most: the API key goes in the headers, the date has to be generated automatically, the arrival time was buried several layers deep in the JSON, and the free tier rate-limited me until I added a pause between calls.

### 4. Local Events (Ticketmaster API)

Finally, upcoming concerts and shows per city — event name, date, venue, and category. The friendliest API of the three, and a clear demand signal: a stadium show means a lot of people gathered in one place.

---

## How I Designed the Database

Instead of one giant messy table, I split the data so it stays clean and keeps history:

cities
|
+---- populations    (population over time)
+---- weather        (5-day forecast per city)
+---- flights        (tomorrow's arrivals)
+---- events         (upcoming concerts and shows)

`cities` holds the **static facts** — one row per city. Everything else links back to it by `city_id`, which is a primary key in `cities` and a foreign key everywhere else. That single link is what lets me join weather and flights for the same city, or keep a population history instead of overwriting it.

- SQL schema: `sql/gans_schema.sql`
- Pipeline notebook: `gans_pipeline.ipynb`

---

## What I Took Away From It

This project pushed me more on the engineering side than the analysis side, and a few lessons really stuck:

- **Every API is different** — key in the URL versus the headers, deeply nested fields, rate limits — so it pays to inspect one raw response *before* writing the parser.
- A good **database design** (static facts versus time-series, linked by a key) keeps everything clean and reusable.
- **Small, composable functions** beat one giant script — far easier to test, debug, and reuse.
- **Real-world data is messy.** Timezones that break MySQL, missing fields, silent rate limits — most of the work was handling the edge cases, not the happy path.

As a student, the biggest win was building something genuinely end-to-end: from raw, scattered sources all the way to a clean, queryable database.

---

## Tools Used

- **Python** (requests, BeautifulSoup, pandas, SQLAlchemy)
- **SQL** (MySQL)
- **APIs** — OpenWeather, AeroDataBox (RapidAPI), Ticketmaster

---

## Getting Started

1. Install the dependencies:
2. Create the database by running `sql/gans_schema.sql` once in your SQL client.
3. Copy `.env.example` to `.env` and add your own API keys and database password.
4. Open `gans_pipeline.ipynb` and run the cells from top to bottom.

All three APIs have free tiers — OpenWeather, AeroDataBox (via RapidAPI), and the Ticketmaster Developer program.

---

## Next Step

Right now I run the notebook by hand. The natural next step is **automation** — deploying the collection functions as a scheduled cloud job so that fresh weather, flights, and events flow into the database every day with no manual steps.

---

## Project Details

- **Cities covered:** 22 European cities (flights and events demoed on a focused subset)
- **Project type:** Data engineering / ETL pipeline
- **Completed:** May 2026
