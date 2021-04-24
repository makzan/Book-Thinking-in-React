React Input and Weather: Part 1 & 2
In this lecture, we make use of the OpenWeatherMap API to fetch data from cities all over the world.

We will create the following result in the end.


Part 1: Fetching Data

We use OpenWeatherAPI.
https://openweathermap.org/api

Before getting started, we want to take a look at the OpenWeatherAPI.


We create our `WeatherData` class to handle the data fetching from the OpenWeatherAPI source.

class WeatherData {
  constructor(cityName="Macao") {
    this.cityName = cityName;
    this.list = [];
    
    // You may need to use your own API key.
    this.apiKey = "9da4ab7247b9bfa529c947d04e087fc3";
    this.apiBase = "https://api.openweathermap.org/data/2.5";    
  }
  
  changeCity(cityName) {
    this.cityName = cityName;    
  }
  
  jsonURL(apiPath) {
    return `${this.apiBase}${apiPath}?units=metric&q=${this.cityName}&APPID=${this.apiKey}`;
  }
  
  fetchCurrent(callback) {
    console.log("Called fetchCurrent")
    var apiPath = "/weather";
    fetch(this.jsonURL(apiPath))
      .then( (response) => {      
        return response.json();
      })
      .then( (data) => {
        this.description = data.weather[0].description;
        callback && callback({
          cityName: this.cityName,
          weatherDescription: this.description,
          lat: data.coord.lat,
          lng: data.coord.lon,
        })
      });
  }
  
  fetchForecast(callback) {   
    console.log("Called fetchForecast")
    var apiPath = "/forecast/daily";
    fetch(this.jsonURL(apiPath))
      .then( (response) => {      
        return response.json();
      })
      .then( (data) => {
        this.list.length = 0;
        for (var forecast of data.list) {
          this.list.push({
            date: new Date(forecast.dt * 1000),
            description: forecast.weather[0].description,
            temperature: forecast.temp.day
          });
        }
        callback && callback({
          cityName: this.cityName,
          forecastList: this.list
        })
      });
  }
}

We make use of the data by initializing our class.

var weatherData = new WeatherData("Macao");

We will define our main component, which fetches data inside `useEffect`.

function WeatherComponent(props) {
  const [cityName, setCityName] = React.useState(props.source.cityName);
  const [weatherDescription, setWeatherDescription] = React.useState("");
  const [lat, setLat] = React.useState("0.00");
  const [lng, setLng] = React.useState("0.00");
  const [forecastList, setForecastList] = React.useState([]);
  
  React.useEffect(() => {
    console.log("Here in useEffect.")
    refreshData(props.source);    
  });
  
  function refreshData(source){
    console.log("Called Refresh Data.")
    var weather = props.source;  
    weather.fetchCurrent( (data) => {
      setCityName(data.cityName);
      setWeatherDescription(data.weatherDescription);
      setLat(data.lat);
      setLng(data.lng);      
    });
    
    weather.fetchForecast( (data) => {
      setCityName(data.cityName);
      setForecastList(data.forecastList);      
    });
  }
  
  return <div>
    <p>{cityName}</p>
    <p>{weatherDescription}</p>
    <p>{lat}, {lng}</p>    
  </div>
}

And of course, we need to mount the WeatherComponent to the DOM element.

// view rendering
ReactDOM.render(
  <WeatherComponent source={weatherData} />,
  document.getElementById('weather-component')
);

Demo:

https://codepen.io/makzan/pen/16ffdfec92a6981aaa1c7daa563f3057


On `useEffect` dependency

The `useEffect` is the side effect code when components are rendered or updated. 

  React.useEffect(() => {
    console.log("Here in useEffect.")
    refreshData(props.source);    
  });

If we take a look at the network activity, we will find that the data fetching loads multiple-time in a short period. That’s because the state changes after the `refreshData` function trigger this side-effect code and create a loop of data refreshment.


CleanShot 2021-04-10 at 12.43.32@2x.png 158 KB


We can fix it by setting the `useEffect` dependency. 
Try to change the `useEffect` code into the following, with a second parameter `[]` as the dependency array.

  React.useEffect(() => {
    console.log("Here in useEffect.")
    refreshData(props.source);    
  }, []);

Now the code only executes right after the component is rendered but not when inner states change.


CleanShot 2021-04-10 at 12.47.33@2x.png 36.5 KB




Part 2: Define UI Placeholder

The UI is divided into the following parts:

  <div>
    <CityInput value={cityName} onChange={handleChange} />
    <WeatherNode cityName={cityName} weather={weatherDescription} />
    <ForecastList forecastList={forecastList} />
    <CityLocation lat={lat} lng={lng} />    
  </div>

Each of them is a placeholder now.

function WeatherNode(props) {
  let {cityName, weather} = props;
  return <p>
    Hello <em>{"CITY_NAME"}</em>. It is <strong>{"WEATHER"}</strong> today.
  </p>
}

function ForecastList(props) {
  let {forecastList} = props;
  return <div>
    <p>7 days forecast:</p>
    <ul>
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
      <li>
        {Math.round(23)}C, {"DATE_HERE"}, {"DESCRIPTION"}
      </li> 
    </ul>
  </div>
}

function CityLocation(props) {
  let {lat, lng} = props;  
  return <div>
    <p>
      <a href="https://www.openstreetmap.org/#map=14/22.200/113.546" target="_blank">(22.200, 113.546)</a>
    </p>
  </div> 
}

Except the `CityInput`, which is a workable code already.

function CityInput(props) {
  const [value, setValue] = React.useState(props.value);  
  
  let defaultCities = ["London", "New York", "Beijing", "Shanghai", "Sydney", "Helsinki"];
  
  let {sampleCities=defaultCities} = props;
  
  function handleTyping(event) {
    setValue(event.target.value);        
    props.onChange && props.onChange(event.target.value);    
  }
  function handleClick(event) {
    setValue(event.target.text)
    props.onChange && props.onChange(event.target.text);
  }
  
  return <div>
    <input type='text' placeholder='City name...' value={value} onChange={handleTyping} />
    <p>Sample:
    {sampleCities.map( city=> 
      <a key={city} className="sample-city" onClick={handleClick}>{city}</a>
    )}         
    </p>
  </div>

We will also handle the input changes in our main WeatherComponent function.

function WeatherComponent(props) {
  const [cityName, setCityName] = React.useState(props.source.cityName);
  const [weatherDescription, setWeatherDescription] = React.useState("");
  const [lat, setLat] = React.useState("0.00");
  const [lng, setLng] = React.useState("0.00");
  const [forecastList, setForecastList] = React.useState([]);
  
  React.useEffect(() => {
    console.log("Here in useEffect.")
    refreshData(props.source);    
  }, []);
  
  function refreshData(source){
    console.log("Called Refresh Data.")
    var weather = props.source;  
    weather.fetchCurrent( (data) => {
      setCityName(data.cityName);
      setWeatherDescription(data.weatherDescription);
      setLat(data.lat);
      setLng(data.lng);      
    });
    
    weather.fetchForecast( (data) => {
      setCityName(data.cityName);
      setForecastList(data.forecastList);      
    });
  }
  
  function handleChange(newCityName) {
    console.log("Handling change to new city name:", newCityName);
    props.source.changeCity(newCityName);    
    refreshData();
  };
  
  return <div>
    <CityInput value={cityName} onChange={handleChange} />
    <WeatherNode cityName={cityName} weather={weatherDescription} />
    <ForecastList forecastList={forecastList} />
    <CityLocation lat={lat} lng={lng} />    
  </div>
}