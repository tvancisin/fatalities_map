<script>
  import { onMount } from "svelte";
  import Map from "./lib/Map.svelte";
  import Visualization from "./lib/Visualization.svelte";
  import * as d3 from "d3";
  import { getGeo, getCSV } from "./utils";

  let vh;
  let width;
  let height;
  let mapRef;
  let correct_height = window.initialHeight;
  let mapLoaded = false;
  let selectedProperties;
  let heading = "Conflict and Peace Process Map";

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

  //load geojson
  let mygeojson;
  const json_path = "/data/country_polygons.json";
  let scaleHeight = d3.scaleLinear().domain([1, 10000]).range([1, 100000]);

  getGeo(json_path).then((geo) => {
    //create array of country codes
    const isoA3Map = polygon_data.reduce((acc, country) => {
      acc[country.iso3c] = country;
      return acc;
    }, {});

    //remove countries not in above array and add fatalities to geojson
    const filteredGeoJSON = {
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

    mygeojson = filteredGeoJSON;
  });

  function handleClose() {
    d3.select(".visualization").style("right", "-100%");
    mapRef.flyToInitialPosition();
    heading = "Conflict and Peace Process Map";
  }

  function handlePolygonClick(event) {
    selectedProperties = event.detail.ADMIN;
    heading = selectedProperties;
    d3.select(".visualization").style("right", "0px");
  }

  let selected_country_details;
  $: if (selectedProperties) {
    selected_country_details = country_data.find(function (d) {
      return d.name == selectedProperties;
    });
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
    <!-- <main bind:clientWidth={width} bind:clientHeight={height}>
  <div role="presentation" id="loading_screen" on:click={handleScreenClick}> -->
    <button id="loading_button" on:click={handleScreenClick}
      >Visualization</button
    >
  </div>
  <h1>{heading}</h1>
  {#if mygeojson && icon_data}
    <Map
      {mygeojson}
      {icon_data}
      bind:this={mapRef}
      on:polygonClick={handlePolygonClick}
      on:mapLoaded={handleMapLoaded}
    />
  {/if}
  <div id="legend">
    <svg height="45px" width="200px">
      <defs>
        <linearGradient id="legend_gradient">
          <stop class="stop_one" offset="0%" />
          <stop class="stop_two" offset="50%" />
          <stop class="stop_three" offset="100%" />
        </linearGradient>
      </defs>
      <rect id="legend_rectangle" x="5" rx="2" y="0" width="200" height="20" />
      <text x="5" y="35" fill="white" font-size="11" text-anchor="start"
        >more fatalities</text
      >
      <text x="200" y="35" fill="white" font-size="11" text-anchor="end"
        >less fatalities</text
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
    background-color: black;
    z-index: 10;
    width: 100vw;
    display: flex;
    justify-content: center;
    /* align-items: center; */
  }

  #loading_button {
    position: absolute;
    top: 40%;
    font-family: "Montserrat";
    background-color: black;
    color: white;
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
    position: absolute;
    font-weight: 400;
    top: 0px;
    font-size: 2em;
    width: 100%;
    margin: 0px;
    text-align: center;
    color: #f0eee3;
    background: rgba(0, 0, 0);
    z-index: 1;
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
      font-size: 1.4em;
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
    stop-color: #E58344;
  }
  .stop_two {
    stop-color: #973102;
  }
  .stop_three {
    stop-color: #4E0303;
  }
</style>
