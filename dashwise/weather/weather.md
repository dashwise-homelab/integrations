# Weather

Weather displays current conditions and a short forecast using Open-Meteo. It maps weather codes to icons and descriptions and includes temperature, precipitation, wind, humidity, sunrise, sunset, and a forecast insight.

## Files

- `integration.yaml` contains the Open-Meteo request, weather-code lookup table, computed fields, and widget configuration.
- `weather.md` documents the integration for contributors and users.

## Configuration

- `WEATHER_LOCATION` — required location object containing a name, latitude, and longitude.
- `UNIT` — optional temperature unit: `c` or `f`.
- `SHOW_LOCATION` — optional toggle for displaying the location name.

Open-Meteo does not require an API token. The response is cached for ten minutes.
