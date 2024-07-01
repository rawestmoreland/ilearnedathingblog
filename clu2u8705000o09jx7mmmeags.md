---
title: "Kubernetes Series Part 1 - Build an Express API"
seoTitle: "Kubernetes Series Part 1 - Build an Express API"
seoDescription: "This will be the beginning of a Kubernetes series that will guide us through deploying a simple Node application to a Kubernetes cluster on DigitalOcean."
datePublished: Fri Oct 28 2022 23:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clu2u8705000o09jx7mmmeags
slug: kubernetes-series-part-1-build-an-express-api
canonical: https://ilearnedathing.com/post/kubernetes-series-part-1-build-an-express-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122437674/bf9af5df-abb8-48a3-8949-890d8c6cebc6.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122442212/b6d12a6e-68cc-49d6-aa3c-acc15a563507.jpeg
tags: docker, javascript, nodejs

---

## What We’ll Build

This will be the beginning of a Kubernetes series that will guide us through deploying a simple Node / Express application to a Kubernetes cluster on DigitalOcean.

Let’s build a simple Express application that queries a REST API. Instead of querying an API directly from you client-side application, we’ll build a small proxy API to stand in the middle of your frontend application and your backend API. Let’s use the [OpenWeatherMap](https://openweathermap.org) API for this project.

## Prerequisites

- Beginner knowledge of REST APIs
- Some knowledge of Express (We won’t go into too much detail on the creation of the Express API)
- Node (v17.5.0+) installed on your machine
- OpenWeatherMap free account and API key

## Get an OpenWeatherMap API Key

Head over to [https://openweathermap.org/api](https://openweathermap.org/api) and sign up for a free account. You’ll be given an API token to use.

<aside>
⚠️ We’ll be using the **2.5 (free)** version of the API and **not** the 3.0 One Call by Call version.

</aside>

## Build the Express App

We’ll use the `Express` framework to build out our api. This framework will start a node server and allow us to create custom routes. Start out by creating a new directory for this project.

```bash
mkdir k8s-series && cd k8s-series
```

Express runs on a node server, so let’s initialize a new npm package for this project.

```bash
# yarn
yarn init -y

# npm
npm init -y
```

For this project, you need a few packages: express and dotenv, and nodemon.

```bash
# npm
npm install express dotenv
npm install -D nodemon

# yarn
yarn add express dotenv
yarn add nodemon -D
```

For nodemon to work, replace your `start` script in your `package.json` with this:

```json
// package.json

"scripts": {
	"start": "nodemon index.js"
}
```

<aside>
⚠️ `node-fetch` is included (experimentally) with node starting in version 17.5. If you are using an older version of node, either switch versions, or install node-fetch@2.6.7

</aside>

Fetching the current weather for our zip code will require two calls to OpenWeather. The first is a call to the GeoLocation API which will turn our zip code into `lat` and `lon`. The second API call will fetch the current weather with the coordinates we receive.

Create a new file called `index.js` in the root of your project’s directory and add the following code.

```jsx
// index.js

const express = require('express');
require('dotenv').config();
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello Weather');
});

// Get the weather for a zip code.
app.get('/weather/zip/:zip/:country?', async (req, res) => {
  // Get the zip from the params
  const { zip } = req.params;

  // If we have a country in the params, use it, else use 'US'
  const country = req.params.country || 'US';

  // Get the lat / lon for the zip
  let latLonRes = await fetch(
    `http://api.openweathermap.org/geo/1.0/zip?zip=${zip},${country}&appid=${process.env.WEATHER_API_KEY}`
  );
  const { lat, lon } = await latLonRes.json();

  // Get the weather for the lat / lon received from the previous request
  const weatherRes = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&appid=${process.env.WEATHER_API_KEY}`
  );
  const weatherJson = await weatherRes.json();

  // Return a 200 response with our weather data as JSON
  res.status(200).json({ data: weatherJson });
});

// Start the app on Port 3000
app.listen(PORT, () => {
  console.log(`Example app listening on port ${PORT}`);
});
```

<aside>
⚠️ Do not commit your API key to github / version control! Create a `.env` file and store your API key there. We will learn how to deal with these ‘secrets’ in Kubernetes in a later article.

Also, you will notice there’s no error handling in the example above. For the sake of simplicity, I left it out. I implore you to implement your own error handling if you would like to do that.

</aside>

Start up your server with `yarn start` or `npm run start`. Head over to a browser and visit `localhost:3000/weather/zip/90210`. You _should_ see the current weather for Beverly Hills as JSON in your browser window.

```json
{
  "data": {
    "coord": { "lon": -118.4065, "lat": 34.0901 },
    "weather": [
      { "id": 800, "main": "Clear", "description": "clear sky", "icon": "01d" }
    ],
    "base": "stations",
    "main": {
      "temp": 15.93,
      "feels_like": 15.93,
      "temp_min": 13.97,
      "temp_max": 17.62,
      "pressure": 1010,
      "humidity": 90
    },
    "visibility": 10000,
    "wind": { "speed": 2.06, "deg": 230 },
    "clouds": { "all": 0 },
    "dt": 1665929250,
    "sys": {
      "type": 2,
      "id": 2012608,
      "country": "US",
      "sunrise": 1665928768,
      "sunset": 1665969509
    },
    "timezone": -25200,
    "id": 5328041,
    "name": "Beverly Hills",
    "cod": 200
  }
}
```

## That’s It!

Next, we’ll look at containerizing this app with Docker.

I encourage you to play around with the API you just built. Check out the OpenWeather API docs and try to pass a city name instead of a zip code. If you’d like more detailed info, consider signing up for their [3.0 OneCall API](https://home.openweathermap.org/subscriptions/billing_info/onecall_30/base?key=base&service=onecall_30). You get 1,000 calls a day for free, but you _must_ give them your payment info, so **be careful**

Enjoy 😃
