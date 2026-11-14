const API_URL = "https://api.open-meteo.com/v1/forecast";
const GEO_URL = "https://geocoding-api.open-meteo.com/v1/search";

const statusText = document.getElementById("status");

const searchBtn = document.getElementById("searchBtn");
const locationBtn = document.getElementById("locationBtn");


// Weather code interpretation
function getWeatherInfo(code) {

    if (code === 0)
        return ["☀️", "Clear Sky"];

    if ([1, 2, 3].includes(code))
        return ["🌤️", "Partly Cloudy"];

    if ([45, 48].includes(code))
        return ["🌫️", "Foggy"];

    if ([51, 53, 55, 56, 57].includes(code))
        return ["🌦️", "Drizzle"];

    if ([61, 63, 65, 66, 67, 80, 81, 82].includes(code))
        return ["🌧️", "Rainy"];

    if ([71, 73, 75, 77, 85, 86].includes(code))
        return ["❄️", "Snow"];

    if ([95, 96, 99].includes(code))
        return ["⛈️", "Thunderstorm"];

    return ["🌡️", "Unknown"];
}


// Get current location
function getLocation() {

    if (!navigator.geolocation) {
        statusText.textContent =
            "Geolocation is not supported by your browser.";
        return;
    }

    statusText.textContent = "Requesting location permission...";

    navigator.geolocation.getCurrentPosition(

        position => {

            const lat = position.coords.latitude;
            const lon = position.coords.longitude;

            loadWeather(lat, lon, "Your Current Location");

        },

        error => {

            statusText.textContent =
                "Location access failed. Please search for a city manually.";

        },

        {
            enableHighAccuracy: true,
            timeout: 10000
        }
    );
}


// Search city
async function searchCity() {

    const city = document.getElementById("cityInput").value.trim();

    if (!city) {
        statusText.textContent = "Please enter a city name.";
        return;
    }

    try {

        statusText.textContent = "Searching location...";

        const response = await fetch(
            `${GEO_URL}?name=${encodeURIComponent(city)}&count=1&language=en&format=json`
        );

        if (!response.ok) throw new Error("Location search failed.");

        const data = await response.json();

        if (!data.results || data.results.length === 0) {
            throw new Error("City not found. Try another name.");
        }

        const place = data.results[0];

        const locationName = `${place.name}, ${place.country}`;

        loadWeather(
            place.latitude,
            place.longitude,
            locationName
        );

    } catch (error) {

        statusText.textContent = error.message;

    }
}


// Fetch weather information
async function loadWeather(lat, lon, locationName) {

    try {

        statusText.textContent = "Loading weather information...";

        const params = new URLSearchParams({

            latitude: lat,
            longitude: lon,

            current: [
                "temperature_2m",
                "relative_humidity_2m",
                "apparent_temperature",
                "is_day",
                "precipitation",
                "weather_code",
                "wind_speed_10m"
            ].join(","),

            hourly: [
                "temperature_2m",
                "relative_humidity_2m",
                "precipitation_probability",
                "weather_code",
                "wind_speed_10m"
            ].join(","),

            forecast_days: "2",
            timezone: "auto"

        });

        const response = await fetch(
            `${API_URL}?${params}`
        );

        if (!response.ok) throw new Error("Weather data unavailable.");

        const data = await response.json();

        displayCurrentWeather(data, locationName);

        displayHourlyForecast(data);

        statusText.textContent = "Weather information updated successfully.";

    } catch (error) {

        statusText.textContent = error.message;

    }
}


// Display current weather
function displayCurrentWeather(data, locationName) {

    const current = data.current;

    const [icon, condition] =
        getWeatherInfo(current.weather_code);

    document.getElementById("cityName").textContent =
        locationName;

    document.getElementById("dateTime").textContent =
        "Local time: " + current.time.replace("T", " ");

    document.getElementById("weatherIcon").textContent =
        icon;

    document.getElementById("temperature").textContent =
        Math.round(current.temperature_2m) + "°C";

    document.getElementById("weatherCondition").textContent =
        condition;

    document.getElementById("feelsLike").textContent =
        Math.round(current.apparent_temperature) + "°C";

    document.getElementById("humidity").textContent =
        current.relative_humidity_2m + "%";

    document.getElementById("wind").textContent =
        current.wind_speed_10m + " km/h";

    document.getElementById("rain").textContent =
        data.hourly.precipitation_probability[
            data.hourly.time.indexOf(current.time)
        ] + "%";

}


// Display next 24 hours
function displayHourlyForecast(data) {

    const container =
        document.getElementById("forecastContainer");

    container.innerHTML = "";

    const hourly = data.hourly;

    // Find current hour in the forecast
    let startIndex = hourly.time.indexOf(data.current.time);

    if (startIndex === -1) {
        startIndex = hourly.time.findIndex(
            time => time >= data.current.time
        );
    }

    if (startIndex === -1) {
        throw new Error("Hourly forecast unavailable.");
    }

    // Get next 24 hourly records
    const next24Hours =
        hourly.time.slice(startIndex, startIndex + 24);

    next24Hours.forEach((time, index) => {

        const i = startIndex + index;

        const [icon, condition] =
            getWeatherInfo(hourly.weather_code[i]);

        const card = document.createElement("div");

        card.className = "hour-card";

        const hour = time.split("T")[1];

        card.innerHTML = `
            <p>${index === 0 ? "Now" : hour}</p>

            <div class="icon">${icon}</div>

            <h3>${Math.round(
                hourly.temperature_2m[i]
            )}°C</h3>

            <p>${condition}</p>

            <small>
                Rain: ${hourly.precipitation_probability[i] ?? 0}%
            </small>

            <br><br>

            <small>
                Humidity: ${hourly.relative_humidity_2m[i]}%
            </small>
        `;

        container.appendChild(card);

    });

}


// Button events
locationBtn.addEventListener("click", getLocation);

searchBtn.addEventListener("click", searchCity);

document.getElementById("cityInput")
    .addEventListener("keydown", event => {

        if (event.key === "Enter") {
            searchCity();
        }

    });
