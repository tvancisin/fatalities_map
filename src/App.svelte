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
    "./data/country_data.csv",
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

  //load_central_points
  let labels_geojson;
  let central_path = "/data/central_points.json";
  getGeo(central_path).then((geo) => {
    labels_geojson = geo;
  });

  //load geojson with all world polygons
  let all_polygons;
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
    d3.select(".visualization").style("right", "-100%");
    d3.select(".information").style("right", "-100%");
    // mapRef.flyToInitialPosition();
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
    <svg height="100px" width="200px">
      <image href="./sign.png" x=62 y=2 height="20" />
      <text
        x="82"
        y="15"
        fill="black"
        font-size="12"
        font-weight="500"
        text-anchor="start">Agreement in 2023</text
      >

      <rect x=40 y=25 width=20 rx=2 height=15 fill="#A1A1A2"/>
      <text
        x="64"
        y="37"
        fill="black"
        font-size="12"
        font-weight="500"
        text-anchor="start">Agreement since 1990</text
      >

      <rect x=5 y=61 width=50 height=5 fill="#CD9E9D"/>
      <rect x=55 y=56 width=50 height=10 fill="#A26F6F"/>
      <rect x=105 y=51 width=50 height=15 fill="#932A2A"/>
      <rect x=155 y=46 width=50 height=20 fill="#311110"/>

      <defs>
        <linearGradient id="legend_gradient">
          <stop class="stop_one" offset="0%" />
          <stop class="stop_two" offset="50%" />
          <stop class="stop_three" offset="100%" />
        </linearGradient>
      </defs>
      <rect id="legend_rectangle" x="5" rx="2" y="68" width="200" height="15" />
      <text
        x="5"
        y="95"
        fill="black"
        font-size="12"
        font-weight="500"
        text-anchor="start">less fatalities</text
      >
      <text
        x="200"
        y="95"
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
    border-radius: 4px;
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
    /* width: 100%; */
    padding: 5px;
    top: -2px;
    font-size: 2em;
    margin: 0px;
    text-align: center;
    color: black;
    background: white;
    z-index: 1;
    transition: top 0.3s ease;
    box-shadow: 0 0 3px #5a5a5a;
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
    width: 205px;
    position: absolute;
    bottom: 5px;
    right: 5px;
    background-color: #ffffffc9;
    border-radius: 3px;
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
    width: 50px;
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
      width: 30px;
    }
  }
</style>
