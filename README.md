# Panahon Ngayon

Panahon Ngayon is a simple weather application built with Vue 3, TypeScript, and Vite. It provides current weather conditions and a 3-day forecast for selected regions, provinces, and cities in the Philippines, along with weather advice and PAGASA advisories.

## Features

*   **Location Selection:** Choose a region, province, and city in the Philippines to get localized weather information.
*   **Current Weather:** Displays current temperature, 'feels like' temperature, humidity, wind speed, and weather conditions.
*   **3-Day Forecast:** Provides a forecast for the next three days, including high/low temperatures and chance of rain.
*   **Weather Advice:** Offers practical advice based on current weather conditions (e.g., what to do during sunny or rainy weather).
*   **PAGASA Advisories:** Includes links and information related to PAGASA (Philippine Atmospheric, Geophysical and Astronomical Services Administration) advisories for LPA, typhoons, and floods.
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