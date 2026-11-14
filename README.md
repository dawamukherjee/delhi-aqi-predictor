# Delhi AQI Predictor: forecast-driven air purifier scheduling

**Track:** AI + Climate
**Team:** Aahan Chaurasia, Tanveer Kaliraman, Dawa Mukherjee

*This project is a prototype and is not medical advice.*

## The issue

Delhi's outdoor PM2.5 changes a lot from every single hour, and a lot of households leave their air purifiers running all day, using up electricity even when the air is actually clean. A family requires to know when to run its purifier and when it's alright to switch it off.

## What we built

We created a machine learning model which reads the last few hours of PM2.5, PM10 and weather (temperature, humidity, wind, rain) for central New Delhi and forecasts PM2.5 **six hours ahead**. A simple rule then switches a purifier ON when the forecast is above **50 µg/m³**, and OFF otherwise. We measure the decisions this leads to: bad-air hours covered, unnecessary running hours, and energy saved.

**User:** a Delhi household with a home air purifier (we assumed a constant 60 W).

## What we added

We built this from the AI + Climate brief (forecasting with the "same as now" baseline) and added our own parts:
- **Our own place and decision:** a Delhi purifier on/off rule, scored on real decisions rather than only forecast error.
- **Our own question:** how does the trigger level trade safety against energy saved? We tested five trigger levels (30 to 70) and chose one.

## How it works

**Data:** hourly PM2.5, PM10, temperature, humidity, wind speed and precipitation for New Delhi (28.6214, 77.2148), 14 Nov 2025 to 9 Nov 2026, 8,664 rows, no missing values or duplicate hours. The air quality values are **Open-Meteo model estimates, not sensor readings.**

**Features:** current PM2.5 and PM10, PM2.5 one and three hours ago, weather, and hour of day.

**Split:** by time, never at random. We trained on older data and tested on the **last 60 days** (1,440 hours), which the model never saw during training.

**Models:** a baseline ("six hours from now will be the same as right now"), a linear regression, and a random forest (100 trees, max depth 12).

## Results

| Model | Average error (µg/m³) | Compared with baseline |
|---|---|---|
| Baseline (same as now) | 22.9 | n/a |
| Linear regression | 18.1 | about 21% better |
| **Random forest (final)** | **16.3** | **about 29% better** |

**Purifier rule.** "Bad air" means PM2.5 above 60 µg/m³ six hours ahead (867 of the 1,440 test hours). We tested five trigger levels for the forecast:

| Switch ON when forecast is above | Bad hours missed | Wasted ON hours | Energy saved vs always on |
|---|---|---|---|
| 30 | 0 | 519 | 4% |
| 40 | 1 | 447 | 9% |
| **50 (our choice)** | **38** | **298** | **22%** |
| 60 | 111 | 155 | 37% |
| 70 | 252 | 79 | 52% |

We chose 50 as a balance: it covers about 96% of bad-air hours (829 of 867) and saves about 19 kWh over 60 days at 60 W. A simple rule that switches ON when the *current* air is above 60 covered only 77% of bad hours (669 of 867), although that was a different trigger, so the comparison is rough.

**Where it fails.** The model struggles with sudden changes. On 17 Oct 2026 it predicted pollution would stay high when the air cleared quickly (off by about 83 µg/m³), and on 3 to 4 Nov it underestimated evening spikes. It only sees current conditions, so it cannot anticipate abrupt shifts. We believe wind or rain caused the Oct 17 clearing, but we did not check this.

## Limitations

- Data is model estimates, not station measurements.
- Forecasts are for **outdoor** air, but a purifier works on **indoor** air. We did not measure indoor air.
- We assumed a constant 60 W and a simple ON/OFF rule.
- We chose the trigger level using the test period, so the results may be a little optimistic. With more time we would pick it on a separate validation period.
- One year of data covers one winter, and PM10 had an extreme value (1,989 µg/m³) that we did not investigate.

## What we'd do next

1. Compare against real sensor readings from a Delhi station.
2. Add an indoor air sensor and fan-speed control, then test in a real home or school.

## Files

- `notebooks/01_data.ipynb`: loading, checking and charting the data
- `notebooks/02_model.ipynb`: baseline, models, failures and the purifier rule
- `delhi_pm25_weather_hourly.csv`: the aligned dataset

## Sources

- Air quality data: Open-Meteo Air Quality API, https://open-meteo.com (CC BY 4.0)
- Weather data: Open-Meteo Historical Weather API, https://open-meteo.com (CC BY 4.0)
- Libraries: scikit-learn, pandas, NumPy, Matplotlib
- AI tools: Claude (Anthropic) was used for guidance, code suggestions and as well as in help drafting this write-up for formatting and more. Our team ran the code and checked the results.
