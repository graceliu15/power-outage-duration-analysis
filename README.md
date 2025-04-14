# title


## Introduction
Have you ever been stuck in a power outage, patiently waiting for the electricity to come back only for DTE’s predictions to be wrong again? What if you could predict how long an outage will last by yourself? As prolonged outages lead to greater disruptions in daily life, it is important energy companies provide consumers with an accurate estimate of how long a power outage will last, so they can prepare, whether it's for a few hours or several days without electricity.

This analysis will explore this dataset which contains climate, geographic, and economic information pertaining to major power outages in the United States from January 2000 – July 2016. This dataset contains 1534 rows (observations) and 55 columns (features). 

## Investigative Question
Specifically, the following analysis will focus on power outages caused by severe weather to answer the following question:
- How does the cause, climate region, and climate category impact the duration of power outages due to severe weather?

## Key Features
The following columns will be used to address this question:
- OUTAGE.DURATION: Duration of outage events (in minutes)
- CAUSE.CATEGORY.DETAIL: Detailed description of the event categories causing the major power outages such as heavy wind, thunderstorm, tornado, etc.
- CLIMATE.REGION: U.S. Climate regions as specified by National Centers for Environmental Information (nine climatically consistent regions in continental U.S.A.). This includes Central, East North Central, Northeast, Northwest, South, Southeast, Southwest, West, and West North Central.
- CLIMATE.CATEGORY: Represents the climate episodes corresponding to the years. The categories—“Warm”, “Cold” or “Normal” episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)
