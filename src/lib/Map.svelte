<script>
  import { onMount, createEventDispatcher } from "svelte";
  import mapboxgl from "mapbox-gl";
  import { all_countries } from "../assets/data/all_countries";
  import * as turf from "turf";

  const dispatch = createEventDispatcher();

  export let mygeojson;
  console.log(mygeojson);

  let map;
  let current_zoom = 2.5;

  function adjustMapForWindowSize() {
    let centerCoordinates = map.getCenter();
    if (window.innerWidth <= 768) {
      current_zoom = 1.4;
      map.flyTo({
        center: [centerCoordinates.lng, centerCoordinates.lat],
        zoom: 1.4,
      });
    } else if (window.innerWidth <= 1000) {
      current_zoom = 2.2;
      map.flyTo({
        center: [centerCoordinates.lng, centerCoordinates.lat],
        zoom: 2.2,
      });
    } else {
      current_zoom = 2.5;
      map.flyTo({
        center: [centerCoordinates.lng, centerCoordinates.lat],
        zoom: 2.5,
      });
    }
  }

  let hoveredPolygonId = null;

  onMount(() => {
    mapboxgl.accessToken =
      "pk.eyJ1Ijoic2FzaGFnYXJpYmFsZHkiLCJhIjoiY2xyajRlczBlMDhqMTJpcXF3dHJhdTVsNyJ9.P_6mX_qbcbxLDS1o_SxpFg";
    map = new mapboxgl.Map({
      container: map,
      style: "mapbox://styles/sashagaribaldy/clxstrxes00qv01pf8dgl4o20",
      center: [50.224518, 22.213995],
      zoom: 2.5,
      maxZoom: 5,
    });

    map.on("load", () => {
      // Variable to hold the ID of the currently hovered feature
      map.addSource("countries", {
        type: "geojson",
        data: mygeojson,
        generateId: true, //This ensures that all features have unique IDs
      });

      map.addLayer({
        id: "3d-buildings",
        type: "fill-extrusion",
        source: "countries",
        paint: {
          "fill-extrusion-color": {
            property: "total_fatalities",
            stops: [
              [100, "#bdbcbc"],
              [10000, "#767676"],
              [100000, "#403F3F"],
            ],
          },
          "fill-extrusion-height": [
            "case",
            ["boolean", ["feature-state", "hover"], false],
            70000, // Height when hovered
            ["get", "total_fatalities"], // Default height
          ],
          "fill-extrusion-base": 0,
          "fill-extrusion-opacity": 1,
        },
      });

      map.addLayer({
        id: "countries-layer",
        type: "fill",
        source: "countries",
        paint: {
          "fill-color": {
            property: "total_fatalities",
            stops: [
              [1, "#BEBEBE"],
              [1000, "#6F4F4F"],
              [100000, "#512B2B"],
            ],
          },
          "fill-opacity": [
            "case",
            ["<", ["get", "number"], 5],
            0,
            [">=", ["get", "number"], 5],
            1,
            1,
          ],
        },
      });

      // // Add outline layer
      // map.addLayer({
      //   id: `borders`,
      //   type: "line",
      //   source: "countries",
      //   paint: {
      //     "line-color": "#000",
      //     "line-width": [
      //       "case",
      //       ["boolean", ["feature-state", "hover"], false],
      //       2,
      //       0,
      //     ],
      //   },
      // });

      map.on("click", "countries-layer", (e) => {
        let clicked_country = e.features[0].properties.ADMIN;
        const properties = e.features[0].properties;
        dispatch("polygonClick", properties);

        let bound_box;
        if (clicked_country == "Russia") {
          bound_box = [
            68.1434025400001, 86.74555084800015, 97.36225305200006,
            35.49540557900009,
          ];
        } else if (clicked_country == "United States of America") {
          bound_box = [
            -160.3688042289999, 24.546282924364334, -36.7005916009999,
            32.71283640500015,
          ];
        } else if (clicked_country == "France") {
          bound_box = [
            -8.691314256999902, 40.909613348000065, 12.771169467000021,
            50.84788646,
          ];
        } else {
          let countries = all_countries.features;
          let the_country = countries.find(function (d) {
            return d.properties.ADMIN == clicked_country;
          });
          bound_box = turf.bbox(the_country);
        }

        if (window.innerWidth <= 768) {
          bound_box[3] -= 10;
        }

        map.fitBounds(bound_box, {
          padding: 50,
        });
      });

      map.on("mousemove", `countries-layer`, (e) => {
        map.getCanvas().style.cursor = "pointer";
        if (e.features.length > 0) {
          if (hoveredPolygonId !== null) {
            map.setFeatureState(
              { source: "countries", id: hoveredPolygonId },
              { hover: false },
            );
          }
          hoveredPolygonId = e.features[0].id;
          map.setFeatureState(
            { source: "countries", id: hoveredPolygonId },
            { hover: true },
          );
        }
      });

      // When the mouse leaves the state-fill layer, update the feature state of the
      // previously hovered feature.
      map.on("mouseleave", `countries-layer`, () => {
        map.getCanvas().style.cursor = "";
        if (hoveredPolygonId !== null) {
          map.setFeatureState(
            { source: "countries", id: hoveredPolygonId },
            { hover: false },
          );
        }
        hoveredPolygonId = null;
      });
    });
    window.addEventListener("load", adjustMapForWindowSize);
    window.addEventListener("resize", adjustMapForWindowSize);
  });

  function flyToInitialPosition() {
    map.flyTo({ center: [50.224518, 22.213995], zoom: current_zoom });
  }

  export { flyToInitialPosition };
</script>

<div id="map" bind:this={map} style="width: 100%; height: 100vh;"></div>

<style>
  div {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
  }
</style>
