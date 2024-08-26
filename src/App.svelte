<script>
  import { onMount } from "svelte";
  import Map from "./lib/Map.svelte";
  import Visualization from "./lib/Visualization.svelte";
  import Info from "./lib/Info.svelte";
  import * as d3 from "d3";
  import { getGeo, getCSV } from "./utils";

  let vh;
  let width;
  let height;
  let mapRef;
  let correct_height = window.initialHeight;
  let mapLoaded = false;
  let selectedProperties;

  //temporary country array
  let ctry_array = [
    {
      country: "Mexico",
      iso3: "MEX",
      coordinates: { lat: 23.6345, lon: -102.5528 },
    },
    {
      country: "Colombia",
      iso3: "COL",
      coordinates: { lat: 4.5709, lon: -74.2973 },
    },
    {
      country: "Venezuela",
      iso3: "VEN",
      coordinates: { lat: 6.4238, lon: -66.5897 },
    },
    {
      country: "Ecuador",
      iso3: "ECU",
      coordinates: { lat: -1.8312, lon: -78.1834 },
    },
    {
      country: "Brazil",
      iso3: "BRA",
      coordinates: { lat: -14.235, lon: -51.9253 },
    },
    {
      country: "Russia",
      iso3: "RUS",
      coordinates: { lat: 61.524, lon: 105.3188 },
    },
    {
      country: "Ukraine",
      iso3: "UKR",
      coordinates: { lat: 48.3794, lon: 31.1656 },
    },
    {
      country: "Armenia",
      iso3: "ARM",
      coordinates: { lat: 40.0691, lon: 45.0382 },
    },
    {
      country: "Azerbaijan",
      iso3: "AZE",
      coordinates: { lat: 40.1431, lon: 47.5769 },
    },
    {
      country: "Mali",
      iso3: "MLI",
      coordinates: { lat: 17.5707, lon: -3.9962 },
    },
    {
      country: "Benin",
      iso3: "BEN",
      coordinates: { lat: 9.3077, lon: 2.3158 },
    },
    {
      country: "Niger",
      iso3: "NER",
      coordinates: { lat: 17.6078, lon: 8.0817 },
    },
    {
      country: "Burkina Faso",
      iso3: "BFA",
      coordinates: { lat: 12.2383, lon: -1.5616 },
    },
    { country: "Togo", iso3: "TGO", coordinates: { lat: 8.6195, lon: 0.8248 } },
    {
      country: "Cameroon",
      iso3: "CMR",
      coordinates: { lat: 7.3697, lon: 12.3547 },
    },
    {
      country: "Nigeria",
      iso3: "NGA",
      coordinates: { lat: 9.082, lon: 8.6753 },
    },
    {
      country: "Central African Republic",
      iso3: "CAF",
      coordinates: { lat: 6.6111, lon: 20.9394 },
    },
    {
      country: "Chad",
      iso3: "TCD",
      coordinates: { lat: 15.4542, lon: 18.7322 },
    },
    {
      country: "Congo, Republic of the",
      iso3: "COG",
      coordinates: { lat: -0.228, lon: 15.8277 },
    },
    {
      country: "Uganda",
      iso3: "UGA",
      coordinates: { lat: 1.3733, lon: 32.2903 },
    },
    {
      country: "Kenya",
      iso3: "KEN",
      coordinates: { lat: -0.0236, lon: 37.9062 },
    },
    {
      country: "Burundi",
      iso3: "BDI",
      coordinates: { lat: -3.3731, lon: 29.9189 },
    },
    {
      country: "Somalia",
      iso3: "SOM",
      coordinates: { lat: 5.1521, lon: 46.1996 },
    },
    {
      country: "Ethiopia",
      iso3: "ETH",
      coordinates: { lat: 9.145, lon: 40.4897 },
    },
    {
      country: "Mozambique",
      iso3: "MOZ",
      coordinates: { lat: -18.6657, lon: 35.5296 },
    },
    {
      country: "Sudan",
      iso3: "SDN",
      coordinates: { lat: 12.8628, lon: 30.2176 },
    },
    {
      country: "South Sudan",
      iso3: "SSD",
      coordinates: { lat: 6.877, lon: 31.307 },
    },
    {
      country: "Iran",
      iso3: "IRN",
      coordinates: { lat: 32.4279, lon: 53.688 },
    },
    {
      country: "Turkey",
      iso3: "TUR",
      coordinates: { lat: 38.9637, lon: 35.2433 },
    },
    {
      country: "Iraq",
      iso3: "IRQ",
      coordinates: { lat: 33.2232, lon: 43.6793 },
    },
    {
      country: "Syria",
      iso3: "SYR",
      coordinates: { lat: 34.8021, lon: 38.9968 },
    },
    {
      country: "Lebanon",
      iso3: "LBN",
      coordinates: { lat: 33.8547, lon: 35.8623 },
    },
    {
      country: "Israel",
      iso3: "ISR",
      coordinates: { lat: 31.0461, lon: 34.8516 },
    },
    {
      country: "Saudi Arabia",
      iso3: "SAU",
      coordinates: { lat: 23.8859, lon: 45.0792 },
    },
    {
      country: "Yemen",
      iso3: "YEM",
      coordinates: { lat: 15.5527, lon: 48.5164 },
    },
    {
      country: "Afghanistan",
      iso3: "AFG",
      coordinates: { lat: 33.9391, lon: 67.71 },
    },
    {
      country: "India",
      iso3: "IND",
      coordinates: { lat: 20.5937, lon: 78.9629 },
    },
    {
      country: "Pakistan",
      iso3: "PAK",
      coordinates: { lat: 30.3753, lon: 69.3451 },
    },
    {
      country: "Myanmar (Burma)",
      iso3: "MMR",
      coordinates: { lat: 21.9162, lon: 95.956 },
    },
    {
      country: "Thailand",
      iso3: "THA",
      coordinates: { lat: 15.87, lon: 100.9925 },
    },
    {
      country: "Philippines",
      iso3: "PHL",
      coordinates: { lat: 12.8797, lon: 121.774 },
    },
    {
      country: "Indonesia",
      iso3: "IDN",
      coordinates: { lat: -0.7893, lon: 113.9213 },
    },
    {
      country: "Papua New Guinea",
      iso3: "PNG",
      coordinates: { lat: -6.3149, lon: 143.9555 },
    },
  ];

  let labels_geojson = {
    type: "FeatureCollection",
    features: ctry_array.map((country) => ({
      type: "Feature",
      properties: {
        country: country.country,
        iso3: country.iso3,
      },
      geometry: {
        type: "Point",
        coordinates: [country.coordinates.lon, country.coordinates.lat],
      },
    })),
  };

  //recalculating heights
  onMount(() => {
    // Set initial vh value based on window.innerHeight
    updateVH();
    // Add a resize event listener to recalculate on orientation change or resize
    window.addEventListener("resize", updateVH);
    // Force a re-render after a short delay to catch any UI shifts
    setTimeout(updateVH, 100);
  });

  function updateVH() {
    vh = window.innerHeight * 0.01;
    document.documentElement.style.setProperty("--vh", `${vh}px`);
  }

  //clicking on screen or button after map is loaded
  function handleMapLoaded() {
    mapLoaded = true;
    document.getElementById("loading_text").style.visibility = "hidden";
    document.getElementById("loading_button").style.visibility = "visible";
  }

  function handleScreenClick(event) {
    // Recalculate width and height
    width = window.innerWidth;
    height = window.innerHeight;

    if (mapLoaded) {
      document.getElementById("loading_screen").style.visibility = "hidden";
      document.getElementById("loading_button").style.visibility = "hidden";
    }
  }

  //load csv files
  let polygon_data;
  let country_data;
  let icon_data;
  let csv_path = [
    "./data/filled_polygon_data_5.csv",
    "./data/country_data_new.csv",
    "./data/agt_point_data.csv",
  ];

  getCSV(csv_path).then((data) => {
    polygon_data = data[0];
    country_data = data[1];

    let icon_geojson = [];
    data[2].forEach(function (d) {
      let to_push = {
        type: "Feature",
        properties: {
          country: d.Country_entity,
          iso: d.iso_code,
          tooltip: d.point_tooltip,
          iconSize: [25, 25],
        },
        geometry: {
          type: "Point",
          coordinates: [+d.central_longitude, +d.central_latitude],
        },
      };
      icon_geojson.push(to_push);
    });

    icon_data = {
      type: "FeatureCollection",
      features: icon_geojson,
    };
  });

  //load geojson with all world polygons
  let all_polygons
  let mygeojson;
  let myallgeojson;
  let country_array = [];
  let country_dropdown;
  const json_path = "/data/country_polygons.json";
  let scaleHeight = d3.scaleLinear().domain([1, 10000]).range([1, 100000]);

  getGeo(json_path).then((geo) => {
    all_polygons = geo;
    //array of fatalities countries
    const isoA3Map = polygon_data.reduce((acc, country) => {
      acc[country.iso3c] = country;
      return acc;
    }, {});

    //remove countries not in above array and add fatalities to geojson
    const fatalities_geojson = {
      ...geo,
      features: geo.features
        .filter((feature) => isoA3Map[feature.properties.ISO_A3]) // Keep only matching features
        .map((feature) => {
          // Add total_fatalities to each feature
          feature.properties.total_fatalities = scaleHeight(
            +isoA3Map[feature.properties.ISO_A3].total_fatalities,
          );
          return feature;
        }),
    };
    mygeojson = fatalities_geojson;

    country_dropdown = country_data.map((country) => country.name);

    //all and fatalities arrays
    const iso3all = country_data.map((country) => country.iso_code);
    const iso3fatal = polygon_data.map((country) => country.iso3c);
    //remove fatalities from all array
    const resultArray = iso3all.filter((iso3) => !iso3fatal.includes(iso3));
    //remove features from all geojson
    const filteredFeatures = geo.features.filter((feature) =>
      resultArray.includes(feature.properties.ISO_A3),
    );

    myallgeojson = {
      type: "FeatureCollection",
      features: filteredFeatures,
    };

    //names of the countries for labels
    country_array = Object.values(isoA3Map).map(
      (country) => country.country_name,
    );
  });

  function dropdownSelection(country) {
    selectedProperties = country.detail;
    d3.select("h1").style("top", "-50px");
    d3.select(".visualization").style("right", "0px");
  }

  function handleClose() {
    d3.select("h1").style("top", "-2px");
    d3.select(".visualization").style("right", "-100%");
    d3.select(".information").style("right", "-100%");
    mapRef.flyToInitialPosition();
  }

  function handlePolygonClick(event) {
    selectedProperties = event.detail;
    d3.select("h1").style("top", "-50px");
    d3.select(".visualization").style("right", "0px");
  }

  let selected_country_details;
  $: if (selectedProperties) {
    selected_country_details = country_data.find(function (d) {
      return d.name == selectedProperties;
    });
  }

  function openInformation() {
    d3.select(".information").style("right", "0px");
    d3.select(".visualization").style("right", "-100%");
  }
</script>

<main
  bind:clientWidth={width}
  bind:clientHeight={height}
  style="height: calc(var(--vh, 1vh) * 100);"
>
  <div
    role="presentation"
    id="loading_screen"
    style="height: calc(var(--vh, 1vh) * 100);"
    on:click={handleScreenClick}
  >
    <button id="loading_button" on:click={handleScreenClick}
      >Visualization</button
    >
    <p id="loading_text">loading...</p>
  </div>

  <h1>Conflict & Peace Process Map</h1>

  <button id="info_button" on:click={openInformation}>
    <i class="fa fa-info"></i>
  </button>

  {#if mygeojson && myallgeojson && icon_data}
    <Map
      {mygeojson}
      {myallgeojson}
      {labels_geojson}
      {icon_data}
      {all_polygons}
      {country_dropdown}
      bind:this={mapRef}
      on:polygonClick={handlePolygonClick}
      on:mapLoaded={handleMapLoaded}
    />
  {/if}

  <Info on:close={handleClose} />

  <div id="legend">
    <svg height="35px" width="200px">
      <defs>
        <linearGradient id="legend_gradient">
          <stop class="stop_one" offset="0%" />
          <stop class="stop_two" offset="50%" />
          <stop class="stop_three" offset="100%" />
        </linearGradient>
      </defs>
      <rect id="legend_rectangle" x="5" rx="2" y="5" width="200" height="15" />
      <text
        x="5"
        y="35"
        fill="black"
        font-size="12"
        font-weight="500"
        text-anchor="start">less fatalities</text
      >
      <text
        x="200"
        y="35"
        fill="black"
        font-size="12"
        font-weight="500"
        text-anchor="end">more fatalities</text
      >
    </svg>
  </div>

  <Visualization {selected_country_details} on:close={handleClose} />
</main>

<style>
  main {
    width: 100vw;
    font-family: "Montserrat";
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #loading_screen {
    position: absolute;
    background-color: #ffffff;
    z-index: 13;
    width: 100vw;
    display: flex;
    justify-content: center;
    /* align-items: center; */
  }

  #loading_text {
    position: absolute;
    top: 40%;
    color: black;
  }

  #loading_button {
    position: absolute;
    top: 40%;
    font-family: "Montserrat";
    background-color: #ffffff;
    color: black;
    border: 1px solid gray;
    border-radius: 2px;
    padding: 10px 15px;
    font-size: 16px;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      color 0.3s ease;
    visibility: hidden;
  }

  #loading_button:hover {
    background-color: steelblue;
    color: black;
  }

  h1 {
    border-radius: 2px;
    position: absolute;
    font-weight: 400;
    width: 100%;
    top: -2px;
    font-size: 2em;
    margin: 0px;
    text-align: center;
    color: black;
    background: white;
    z-index: 1;
    transition: top 0.3s ease;
  }

  @media only screen and (max-width: 1450px) {
    h1 {
      font-size: 1.8em;
    }
  }

  @media only screen and (max-width: 1200px) {
    h1 {
      font-size: 1.6em;
    }
  }

  @media only screen and (max-width: 768px) {
    h1 {
      font-size: 1.2em;
    }
  }

  #legend {
    position: absolute;
    bottom: 5px;
    right: 10px;
  }

  #legend_rectangle {
    fill: url(#legend_gradient);
  }

  .stop_one {
    stop-color: #e3b5b5;
  }
  .stop_two {
    stop-color: #7e4949;
  }
  .stop_three {
    stop-color: #290a0a;
  }

  #info_button {
    position: absolute;
    top: 3px;
    left: 5px;
    background-color: white;
    border: 1px solid rgb(173, 173, 173);
    color: black;
    padding: 5px 20px;
    text-align: center;
    text-decoration: none;
    border-radius: 2px;
    display: inline-block;
    font-size: 1em;
    cursor: pointer;
    z-index: 10;
  }

  #info_button:hover {
    background-color: #aa4197;
    color: white;
  }

  @media only screen and (max-width: 768px) {
    #info_button {
      font-size: 0.7em;
    }
  }

  @media only screen and (max-width: 500px) {
    #info_button {
      left: 1px;
      padding: 3px 12px;
      font-size: 0.6em;
    }
  }
</style>
