<script>
  import Map from "./lib/Map.svelte";
  import Visualization from "./lib/Visualization.svelte";
  import * as d3 from "d3";
  import { getGeo, getCSV } from "./utils";

  let width, height;

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

  let mapRef;
  let selectedProperties;
  let heading = "Conflict and Peace Process Map"

  function handleClose() {
    d3.select(".visualization").style("right", "-100%");
    mapRef.flyToInitialPosition();
    heading = "Conflict and Peace Process Map"
  }

  function handlePolygonClick(event) {
    selectedProperties = event.detail.ADMIN;
    heading = selectedProperties
    d3.select(".visualization").style("right", "0px");
  }

  let selected_country_details;
  $: if (selectedProperties) {
    selected_country_details = country_data.find(function (d) {
      return d.name == selectedProperties;
    });
  }
</script>

<main bind:clientWidth={width} bind:clientHeight={height}>
  <h1>{heading}</h1>
  {#if mygeojson && icon_data}
    <Map {mygeojson} {icon_data} bind:this={mapRef} on:polygonClick={handlePolygonClick} />
  {/if}
  <Visualization {selected_country_details} on:close={handleClose} />
</main>

<style>
  main {
    width: 100vw;
    height: 100vh;
    font-family: "Montserrat";
  }

  h1 {
    position: absolute;
    top: 0px;
    font-size: 2em;
    width: 100%;
    margin: 0px;
    text-align: center;
    color: white;
    z-index: 999;
    background: rgba(0, 0, 0, 0.9);
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
</style>
