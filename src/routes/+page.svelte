<script>
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
    
    let weatherData = null;
    let error = null;
    let selectedDay = null;
    let searchQuery = '';
    let location = 'Venice, Italy'; // Default location
    let searchResults = [];
    let showDropdown = false;
    let typingTimer;
    const typingDelay = 1000; // 1 second

    async function fetchWeather(lat = 45.490292, lon = 12.168120) {
        try {
            const response = await fetch(
                `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=temperature_2m_max,weathercode&hourly=temperature_2m,weathercode`
            );
            if (!response.ok) throw new Error('Failed to fetch weather data');
            const data = await response.json();
            weatherData = data;
        } catch (err) {
            error = err.message;
        }
    }

    async function fetchCoordinates(query) {
        try {
            const response = await fetch(
                `https://geocode.maps.co/search?q=${encodeURIComponent(query)}&api_key=6646209694390873982100hrc8ef581`
            );
            if (!response.ok) throw new Error('Failed to fetch location data');
            const data = await response.json();
            searchResults = data.map(result => ({
                displayName: result.display_name,
                lat: result.lat,
                lon: result.lon
            }));
            showDropdown = true;
        } catch (err) {
            error = err.message;
        }
    }

    async function handleSelectLocation(lat, lon, displayName) {
        location = displayName;
        showDropdown = false;
        fetchWeather(lat, lon);
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
            99: 'Thunderstorm with heavy hail',
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
            99: '⛈️',
        };
        return weatherIcons[code] || '❓';
    }

    function formatDate(timestamp) {
        const date = new Date(timestamp);
        const options = { weekday: 'short', day: 'numeric', month: 'short', year: 'numeric' };
        return date.toLocaleDateString(undefined, options).toUpperCase();
    }

    function selectDay(index) {
        selectedDay = selectedDay === index ? null : index;
    }

    function handleSearchInput(event) {
        clearTimeout(typingTimer);
        searchQuery = event.target.value;
        typingTimer = setTimeout(() => fetchCoordinates(searchQuery), typingDelay);
    }

    function handleSearchIconClick() {
        clearTimeout(typingTimer);
        fetchCoordinates(searchQuery);
    }

    $: truncatedLocation = location.length > 25 ? `${location.substring(0, 25)}...` : location;
</script>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');
    @import url('https://fonts.googleapis.com/css2?family=Oswald:wght@200..700&display=swap');
    @import url('https://fonts.googleapis.com/css2?family=Abril+Fatface&family=Oswald:wght@200..700&display=swap');
    @import url('https://fonts.googleapis.com/css2?family=Abril+Fatface&family=Oswald:wght@200..700&family=Teko:wght@300..700&display=swap');
    body {
        background-color: #eee2e2;
        font-family: 'Poppins', sans-serif;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        width: 100vw;
        height: 100vh;
        margin: 0;
    }

    .weather-container {
        background: #f7f5f5;
        width: 80%;
        height: 80%;
        border-radius: 25px;
        box-shadow: 0 20px 70px rgba(0, 0, 0, 0.25);
        display: flex;
        flex-direction: row;
        overflow: hidden;
    }

    .weather-list {
        width: 35%;
        display: flex;
        flex-direction: column;
        padding: 20px;
        background-color: #f7f5f5;
        overflow-y: auto;
    }

    .weather-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 16px 0;
        font-size: 16px;
        font-weight: 600;
        color: #595959;
        cursor: pointer;
        transition: background 0.3s;
        border-bottom: 1px solid #e0e0e0;
    }

    .weather-item:hover {
        background-color: #e0e4e8;
    }

    .weather-icon {
        font-size: 1.5em;
        margin-right: 10px;
    }

    .weather-info {
        width: 65%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        padding: 50px;
        position: relative;
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        border-radius: 25px;
        overflow: hidden;
        box-shadow: 0 10px 20px rgba(0, 0, 0, 0.35);
    }

    .weather-info::after {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(255, 255, 255, 0.5);
        z-index: 0;
    }

    .weather-container-circle{
        background: linear-gradient(90deg, rgba(236,215,215,1) 30%, rgba(250,244,244,1) 60%, rgba(255,255,255,1) 100%);
        border-radius: 50%;
        width: 450px;
        height: 450px;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        box-shadow: 0 10px 20px rgba(0, 0, 0, 0.35);
        z-index: 100;
    }
    
    .weather-container-circle h1 {
        font-size: 5em;
        margin: 10px 0;
        color: #333;
        font-weight: 700;
    }

    .weather-container-circle p {
        font-family: "Oswald", sans-serif;
        font-optical-sizing: auto;
        font-weight: 500;
        font-size: 1.7em;
        margin: 5px 0;
        color: #666;
    }

    .search-place {
        position: absolute;
        display: flex;
        top: 20px;
        right: 20px;
        align-items: center;
        background-color: #ecd7d7;
        box-shadow: 0 5px 10px rgba(0, 0, 0, 0.25);
        padding: 10px 20px;
        border-radius: 30px;
        cursor: pointer;
        z-index: 1000;
    }

    .search-place img {
        cursor: pointer;
    }

    .search-place input {
        background-color: #f7f5f5 !important;
        padding: 10px;
        border: none;
        border-radius: 20px;
        margin-left: 10px;
        box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
        font-size: 1em;
        outline: none;
        background: none;
    }

    .dropdown {
        position: absolute;
        top: 100%;
        left: 0;
        right: 0;
        background-color: white;
        border: 1px solid #ccc;
        border-radius: 4px;
        max-height: 200px;
        overflow-y: auto;
        z-index: 1000;
    }
    
    .dropdown li {
        padding: 10px;
        cursor: pointer;
        transition: background-color 0.2s;
        z-index: 1000;
        opacity: 1;
    }

    .dropdown li:hover {
        background-color: #f0f0f0;
    }
    
    .error {
        color: red;
        font-size: 1.5em;
    }

    .hourly-details {
        margin-top: 10px;
        font-size: 14px;
        color: #666;
    }

    .hourly-details table {
        width: 100%;
        border-collapse: collapse;
    }

    .hourly-details tr {
        border-bottom: 1px solid #e0e0e0;
    }

    .hourly-details td {
        padding: 10px 5px;
        text-align: center;
    }

    .iconSize {
        width: 20px;
        height: 20px;
    }
    
    .gifAnimation{
        margin-top: 20px;
        margin: auto;
        width: 100px;
        height: 100px;
        z-index: 100;
    }
    .title{
        font-family: 'Abril Fatface', cursive;
        font-size: 3em;
        color: #333;
        margin-top: -50px;
        margin-bottom: 20px;
    }
</style>

<body>
<h1 class="title">The Weather.</h1>
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
                        <table>
                            {#each weatherData.hourly.temperature_2m.filter((_, hourIndex) =>
                                new Date(weatherData.hourly.time[hourIndex]).toDateString() === new Date(weatherData.daily.time[index]).toDateString()
                            ).slice(0, 24) as temp, hourIndex}
                                <tr>
                                    <td>{new Date(weatherData.hourly.time[hourIndex]).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</td>
                                    <td>{temp}°</td>
                                    <td>{getWeatherIcon(weatherData.hourly.weathercode[hourIndex])}</td>
                                    <td>{getWeatherDescription(weatherData.hourly.weathercode[hourIndex])}</td>
                                </tr>
                            {/each}
                        </table>
                    </div>
                {/if}
            {/each}
            <img src="weather.gif"  class="gifAnimation"  alt="weather"/>
        </div>
        <div class="weather-info" style="background-image: url('/weather/{weatherData.daily.weathercode[0]}.jpg');">
            <div class="search-place">
                <img src="search-icon2.png" class="iconSize" alt="Search Icon" on:click={handleSearchIconClick} />
                <input
                        type="text"
                        placeholder="search place"
                        bind:value={searchQuery}
                        on:input={handleSearchInput}
                />
                {#if showDropdown && searchResults.length > 0}
                    <ul class="dropdown">
                        {#each searchResults as result}
                            <li on:click={() => handleSelectLocation(result.lat, result.lon, result.displayName)}>
                                {result.displayName}
                            </li>
                        {/each}
                    </ul>
                {/if}
            </div>
            <div class="weather-container-circle" >
                <h1>{weatherData.daily.temperature_2m_max[0]}°</h1>
                <p>{getWeatherDescription(weatherData.daily.weathercode[0])}</p>
                <p>{formatDate(weatherData.daily.time[0])}</p>
                <p>{truncatedLocation}</p>
            </div>
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
</div>
</body>