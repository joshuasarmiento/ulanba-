# Ulan Ba Dyan?

Ulan Ba Dyan? is a simplified weather monitoring application built with Vue 3, TypeScript, and Vite. It helps communities stay informed, prepared, and safe by providing accurate weather forecasts and advisories for the Philippines.

## Features

*   **Intuitive Location Search:** Easily find any city in the Philippines using a single search input with autocomplete. Defaults to showing Metro Manila cities when empty.
*   **"Use My Location" Feature:** Get instant weather updates for your current, precise location using the browser's Geolocation API.
*   **Current Weather:** Displays real-time temperature, 'feels like' temperature, humidity, wind speed, and weather conditions.
*   **3-Day Forecast:** Provides a detailed forecast for the next three days, including high/low temperatures and chance of rain, along with a random weather tip for the next day.
*   **Hourly Forecast:** View hourly weather predictions for the selected city.
*   **Weather Advice:** Offers practical advice based on current weather conditions (e.g., what to do during sunny or rainy weather).
*   **LPA & Typhoon Advisories:** Displays real-time weather alerts and advisories, including LPA and Typhoon updates, refreshed hourly.
*   **PAGASA Synopsis:** Includes official PAGASA weather synopsis and links to their advisories.
*   **Dynamic Weather Icons:** Uses animated weather icons from Makin-Things/weather-icons.

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

### Lint with ESLint

```sh
npm run lint
```

## API Key

This application uses the WeatherAPI.com API. You need to sign up at [https://www.weatherapi.com](https://www.weatherapi.com) to get your API key. Once you have the key, create a `.env` file in the root directory of the project and add your API key:

```
VITE_WEATHER_API_KEY=your_api_key_here
```