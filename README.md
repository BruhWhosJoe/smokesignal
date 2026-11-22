# SmokeSignal: Should tomorrow's practice be outdoors?

**Excellence Award, Veritas AI International High School Hackathon 2026** (1,000+ projects from 75+ countries)

**Track:** AI + Climate (built on suggested project brief 4.3.2, Next-day air quality forecast)
**Team:** Jonathan Wang (solo)

**Run it:** [Open in Google Colab](https://colab.research.google.com/drive/1emm4f8--O7a_WYnp9VjEMsoI1Mgtr4Rv), then Runtime → Run all. The first run downloads about 75 MB of EPA data and takes a few minutes. No API keys needed.
**Demo video:** [`SmokeSignal_demo.mp4`](SmokeSignal_demo.mp4) (1:52)

## The problem

In June 2023, smoke from Canadian wildfires made the air around DC unhealthy for several days, and schools had to decide on short notice whether to cancel outdoor sports. Children and people with asthma are among the groups the EPA says are most sensitive to fine-particle pollution (PM2.5). SmokeSignal is for an athletic director at a Montgomery County, Maryland high school who is setting tomorrow's practice plan at about 3 PM.

## What I built and added

You give it today's PM2.5 in Rockville, today's PM2.5 at monitors upwind in Pennsylvania and New York, and today's weather. It gives back a call, either "Outdoor practice OK" or "Plan indoors", and the reason. For example: "the upwind monitors in PA/NY are 29 µg/m³ dirtier than Rockville today". The line it uses is the EPA's 35.4 µg/m³ daily level, where the air becomes "Unhealthy for Sensitive Groups".

I started from the suggested project 4.3.2 (next-day air quality forecast). I added my own user and place (an athletic director in Montgomery County, where I live), my own question (does the forecast still work on wildfire smoke days?), and the upwind smoke watch described below.

## How it works

The brief suggests OpenAQ, but I used the EPA's AirData files instead. They don't need an API key, so the notebook runs for anyone, and one file per year covers every monitor I needed in Maryland, Pennsylvania and New York.

The data is the EPA's daily PM2.5 files for 2017–2025 (3,287 days) plus Open-Meteo's historical weather for Rockville. From the EPA files I built three daily numbers: Rockville (what I forecast), the DC–Baltimore average (to fill gaps), and an "upwind north" average over Pennsylvania and upstate New York.

I trained on 2017–2022 and tested on 2023–2025, without shuffling days. I compared a baseline ("tomorrow = today"), linear regression, and gradient-boosted trees that also get the upwind readings.

My own addition is a smoke watch on top of the model. It warns when the upwind monitors are more than 9.8 µg/m³ dirtier than Rockville. That number is the 99.5th percentile of the gap in the training years.

## Results (test years 2023–2025)

| | Average error (MAE) | Unhealthy days caught | Extra warnings per year |
|---|---|---|---|
| Baseline: tomorrow = today | 2.43 µg/m³ | 1 of 3 | 0.7 |
| Model 1: linear regression | 2.14 µg/m³ | 1 of 3 | 0.7 |
| Model 2: boosted trees + upwind | 2.12 µg/m³ | 0 of 3 | 0 |
| **SmokeSignal (Model 2 + smoke watch)** | **2.12 µg/m³** | **3 of 3** | **5.0** |

Both models beat the baseline by about 12–13% on average. But they missed the days that matter. On the two days right before the smoke arrived, they forecast 14–19 µg/m³ and the real values were 57 and 76. The baseline and Model 1 only caught the one day when the smoke was already here.

I think this happened because the training years had zero days above 35.4, so the models had never seen anything like June 2023. When I gave the linear model the upwind reading, the weight it learned for it was about zero. The pattern just wasn't in the training data.

The smoke watch works because the smoke reaches Pennsylvania and New York about a day before Rockville. On June 6, 2023 the upwind monitors read 52 while Rockville read 24, and Rockville hit 57 the next day. It caught all 3 unhealthy days, at a cost of about 5 extra warnings a year.

However, what makes this less clean is how there are only 3 unhealthy days, all from one smoke season. I got the idea for the rule by looking at June 2023, which is in the test years. And I tried two percentiles (99 and 99.5) before keeping 99.5. The threshold itself only uses training data, but the fair test is the next smoke season. I also use full-day averages for "today", when at 3 PM the day isn't over.

## What I'd do next

First I'd run the same rule for other cities that got the 2023 smoke, like Philadelphia and New York, to see if it holds up on more events. Then I'd pick the upwind monitors using the wind forecast, so it isn't limited to smoke coming from the north.

## What's in this repo

| File | What it is |
|---|---|
| `SmokeSignal.ipynb` | The full notebook: data loading, models, smoke watch, demo and optional Gradio app |
| `data/pm25_daily.csv` | Daily PM2.5 for Rockville, DC–Baltimore and upwind north, built from the EPA files |
| `data/weather_daily.csv` | Daily weather for Rockville from Open-Meteo |
| `SmokeSignal_demo.mp4` | 2-minute demo video |

If the two CSVs are in `data/`, the notebook uses them and skips the download. Delete them to rebuild everything from the original sources.

## Sources

- EPA AirData daily PM2.5 files: https://aqs.epa.gov/aqsweb/airdata/download_files.html
- Open-Meteo Historical Weather API: https://open-meteo.com/en/docs/historical-weather-api (weather data from Open-Meteo.com, used under CC BY 4.0)
- EPA fact sheet on the 2024 AQI breakpoints for PM2.5: https://www.epa.gov/system/files/documents/2024-02/pm-naaqs-air-quality-index-fact-sheet.pdf
- Libraries: pandas, NumPy, scikit-learn, Matplotlib, requests, Gradio (optional)
- AI tools: I used Claude (Opus 5.5 and Fable 5.2). I chose the climate track, and Claude suggested the project idea and found the EPA data files. Claude wrote most of the code, did the debugging and drafted the text. I ran everything myself, checked the outputs, and edited the explanations and write-up.

## License

Code is under the MIT License (see `LICENSE`). The data comes from the U.S. EPA and Open-Meteo, under their own terms.
