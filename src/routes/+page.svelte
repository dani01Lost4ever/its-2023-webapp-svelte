<script>
    import { onMount } from 'svelte';

    let weatherData = null;
    let error = null;

    async function fetchWeather() {
        try {
            const response = await fetch('https://api.open-meteo.com/v1/forecast?latitude=45.490292&longitude=12.168120&hourly=temperature_2m,weathercode');
            if (!response.ok) throw new Error('Failed to fetch weather data');
            const data = await response.json();
            weatherData = data.hourly;
        } catch (err) {
            error = err.message;
        }
    }

    onMount(() => {
        fetchWeather();
    });

    function getWeatherDescription(code) {
        const weatherCodes = {
            0: 'Clear sky',
            1: 'Mainly clear',
            2: 'Partly cloudy',
            3: 'Overcast',
            45: 'Fog',
            48: 'Depositing rime fog',
            51: 'Drizzle: Light',
            53: 'Drizzle: Moderate',
            55: 'Drizzle: Dense intensity',
            56: 'Freezing Drizzle: Light',
            57: 'Freezing Drizzle: Dense intensity',
            61: 'Rain: Slight',
            63: 'Rain: Moderate',
            65: 'Rain: Heavy intensity',
            66: 'Freezing Rain: Light',
            67: 'Freezing Rain: Heavy intensity',
            71: 'Snow fall: Slight',
            73: 'Snow fall: Moderate',
            75: 'Snow fall: Heavy intensity',
            77: 'Snow grains',
            80: 'Rain showers: Slight',
            81: 'Rain showers: Moderate',
            82: 'Rain showers: Violent',
            85: 'Snow showers: Slight',
            86: 'Snow showers: Heavy',
            95: 'Thunderstorm: Slight or moderate',
            96: 'Thunderstorm with slight hail',
            99: 'Thunderstorm with heavy hail'
        };
        return weatherCodes[code] || 'Unknown';
    }

    function getWeatherIcon(code) {
        const weatherIcons = {
            0: '☀️',
            1: '🌤️',
            2: '⛅',
            3: '☁️',
            45: '🌫️',
            48: '🌫️',
            51: '🌧️',
            53: '🌧️',
            55: '🌧️',
            56: '🌧️',
            57: '🌧️',
            61: '🌦️',
            63: '🌦️',
            65: '🌧️',
            66: '🌧️',
            67: '🌧️',
            71: '❄️',
            73: '❄️',
            75: '❄️',
            77: '❄️',
            80: '🌧️',
            81: '🌧️',
            82: '🌧️',
            85: '❄️',
            86: '❄️',
            95: '⛈️',
            96: '⛈️',
            99: '⛈️'
        };
        return weatherIcons[code] || '❓';
    }

    function formatDateTime(timestamp) {
        const date = new Date(timestamp);
        return date.toLocaleString([], { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' });
    }
</script>

<style>
    .weather-container {
        font-family: Arial, sans-serif;
        padding: 25px;
        width: 70%;
        margin: auto;
        background-color: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }
    .error {
        color: red;
    }
    .weather-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
        gap: 10px;
        margin-top: 20px;
    }
    .weather-item {
        background-color: #fff;
        padding: 10px;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        text-align: center;
    }
    .weather-item h3 {
        margin: 0;
        font-size: 0.75em;
        color: #333;
    }
    .weather-item p {
        margin: 5px 0 0;
        font-size: 1.1em;
        color: #007BFF;
    }
    .app {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
    }

    main {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 1rem;
        width: 100%;
        max-width: 64rem;
        margin: 0 auto;
        box-sizing: border-box;
    }

    footer {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        padding: 12px;
    }

    footer a {
        font-weight: bold;
    }

    @media (min-width: 480px) {
        footer {
            padding: 12px 0;
        }
    }
    section {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        flex: 0.6;
    }

    h1 {
        color: #007BFF;
        width: 100%;
    }
    main {
        text-align: center;
        padding: 1em;
        max-width: 800px;
        margin: 0 auto;
        font-family: Arial, sans-serif;
    }
    .welcome {
        display: block;
        position: relative;
        width: 100%;
        height: 0;
        padding: 0 0 calc(100% * 495 / 2048) 0;
    }

    .welcome img {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        display: block;
    }
    @import '@fontsource/fira-mono';

    :root {
        --font-body: Arial, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
        Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
        --font-mono: 'Fira Mono', monospace;
        --color-bg-0: rgb(202, 216, 228);
        --color-bg-1: hsl(209, 36%, 86%);
        --color-bg-2: hsl(224, 44%, 95%);
        --color-theme-1: #ff3e00;
        --color-theme-2: #4075a6;
        --color-text: rgba(0, 0, 0, 0.7);
        --column-width: 42rem;
        --column-margin-top: 4rem;
        font-family: var(--font-body);
        color: var(--color-text);
    }

    body {
        min-height: 100vh;
        margin: 0;
        background-attachment: fixed;
        background-color: var(--color-bg-1);
        background-size: 100vw 100vh;
        background-image: radial-gradient(
                50% 50% at 50% 50%,
                rgba(255, 255, 255, 0.75) 0%,
                rgba(255, 255, 255, 0) 100%
        ),
        linear-gradient(180deg, var(--color-bg-0) 0%, var(--color-bg-1) 15%, var(--color-bg-2) 50%);
    }

    h1,
    h2,
    p {
        font-weight: 400;
    }

    p {
        line-height: 1.5;
    }

    a {
        color: var(--color-theme-1);
        text-decoration: none;
    }

    a:hover {
        text-decoration: underline;
    }

    h1 {
        font-size: 2rem;
        text-align: center;
    }

    h2 {
        font-size: 1rem;
    }

    pre {
        font-size: 16px;
        font-family: var(--font-mono);
        background-color: rgba(255, 255, 255, 0.45);
        border-radius: 3px;
        box-shadow: 2px 2px 6px rgb(255 255 255 / 25%);
        padding: 0.5em;
        overflow-x: auto;
        color: var(--color-text);
    }

    .text-column {
        display: flex;
        max-width: 48rem;
        flex: 0.6;
        flex-direction: column;
        justify-content: center;
        margin: 0 auto;
    }

    input,
    button {
        font-size: inherit;
        font-family: inherit;
    }

    button:focus:not(:focus-visible) {
        outline: none;
    }

    @media (min-width: 720px) {
        h1 {
            font-size: 2.4rem;
        }
    }

    .visually-hidden {
        border: 0;
        clip: rect(0 0 0 0);
        height: auto;
        margin: 0;
        overflow: hidden;
        padding: 0;
        position: absolute;
        width: 1px;
        white-space: nowrap;
    }

</style>

<div class="weather-container">
    {#if error}
        <p class="error">{error}</p>
    {:else if weatherData}
        <h1>Weather Data</h1>
        <div class="weather-grid">
            {#each weatherData.temperature_2m as temp, index}
                <div class="weather-item">
                    <h3>{formatDateTime(weatherData.time[index])}</h3>
                    <p>{temp}°C</p>
                    <p style="font-size: 1.8em">{getWeatherIcon(weatherData.weathercode[index])}</p>
                    <p style="font-size: 0.9em">{getWeatherDescription(weatherData.weathercode[index])}</p>

                </div>
            {/each}
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
</div>
