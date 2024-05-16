<script>
    import { onMount } from 'svelte';

    let weatherData = null;
    let error = null;
    let selectedDay = null;

    async function fetchWeather() {
        try {
            const response = await fetch(
                'https://api.open-meteo.com/v1/forecast?latitude=45.490292&longitude=12.168120&daily=temperature_2m_max,weathercode&hourly=temperature_2m,weathercode'
            );
            if (!response.ok) throw new Error('Failed to fetch weather data');
            const data = await response.json();
            weatherData = data;
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

    function formatDate(timestamp) {
        const date = new Date(timestamp);
        const options = { weekday: 'short', day: 'numeric', month: 'short' };
        return date.toLocaleDateString(undefined, options).toUpperCase();
    }

    function selectDay(index) {
        selectedDay = selectedDay === index ? null : index;
    }
</script>

<style>
    body {
        background-color: #F0F4F8;
        font-family: 'Poppins', sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
    }

    .weather-container {
        background-color: #FFFFFF;
        width: 850px;
        height: 550px;
        border-radius: 15px;
        box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
        display: flex;
        flex-direction: row;
        overflow: hidden;
    }

    .weather-list {
        width: 35%;
        display: flex;
        flex-direction: column;
        padding: 15px;
        background-color: #FAF9F9;
        border-right: 1px solid #E0E0E0;
        overflow-y: auto;
    }

    .weather-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px 0;
        font-size: 14px;
        color: #595959;
        cursor: pointer;
        transition: background 0.3s;
        border-bottom: 1px solid #E0E0E0;
    }

    .weather-item:hover {
        background-color: #F0F4F8;
    }

    .weather-item span {
        display: inline-block;
        margin-right: 5px;
    }

    .weather-icon {
        font-size: 1.2em;
    }

    .weather-info {
        width: 65%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        background-color: white;
        padding: 20px;
        position: relative;
    }

    .weather-info h1 {
        font-size: 4em;
        margin: 5px 0;
        color: #007bff;
    }

    .weather-info p {
        font-size: 1em;
        margin: 5px 0;
        color: #333;
    }

    .search-place {
        position: absolute;
        top: 15px;
        right: 15px;
        display: flex;
        align-items: center;
    }

    .search-place input {
        padding: 8px 15px;
        border: 1px solid #E0E0E0;
        border-radius: 20px;
        margin-left: 10px;
        box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
        font-size: 1em;
        outline: none;
    }

    .hourly-details {
        margin-top: 10px;
        font-size: 12px;
        color: #666;
    }

    .hourly-details p {
        display: flex;
        justify-content: space-between;
        margin: 5px 0;
    }
    .iconSize{
        width: 20px;
        height: 20px;
    }
</style>

<body>
<div class="weather-container">
    {#if error}
        <p class="error">{error}</p>
    {:else if weatherData}
        <div class="weather-list">
            {#each weatherData.daily.temperature_2m_max as temp, index}
                <div class="weather-item" on:click={() => selectDay(index)}>
                    <span>{formatDate(weatherData.daily.time[index])}</span>
                    <span class="weather-icon">{getWeatherIcon(weatherData.daily.weathercode[index])}</span>
                    <span>{temp}°</span>
                </div>
                {#if selectedDay === index}
                    <div class="hourly-details">
                        {#each weatherData.hourly.temperature_2m
                            .filter((_, hourIndex) =>
                                new Date(weatherData.hourly.time[hourIndex]).toDateString() === new Date(weatherData.daily.time[index]).toDateString()
                            )
                            .slice(0, 24) as temp, hourIndex}
                            <p>
                                <span>{new Date(weatherData.hourly.time[hourIndex]).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</span>
                                <span>{temp}°</span>
                                <span>{getWeatherIcon(weatherData.hourly.weathercode[hourIndex])}</span>
                                <span>{getWeatherDescription(weatherData.hourly.weathercode[hourIndex])}</span>
                            </p>
                        {/each}
                    </div>
                {/if}
            {/each}
        </div>
        <div class="weather-info">
            <div class="search-place">
                <img src="search-icon.png" class="iconSize" alt="Search Icon" />
                <input type="text" placeholder="search place" />
            </div>
            <h1>{weatherData.daily.temperature_2m_max[0]}°</h1>
            <p>{getWeatherDescription(weatherData.daily.weathercode[0])}</p>
            <p>{formatDate(weatherData.daily.time[0])}</p>
            <p>Seoul, South Korea</p>
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
</div>
</body>