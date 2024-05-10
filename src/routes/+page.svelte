<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import Switch from "./Switch.svelte";
    import Bargraphs_corp from "./Bargraph_corp.svelte";
    import Bargraphs_own from "./Bargraph_own.svelte";
    let map;
    let toggle;
    let timeFilter = 2005;
    let mounted = false;
    let corpowner_paint;
    $: corpowner_paint = {
      "fill-color": [
        "interpolate",
        ["linear"],
        ["get", "Y" + timeFilter],
        0.1,
        "#F2F12D",
        0.15,
        "#EED322",
        0.2,
        "#E6B71E",
        0.25,
        "#DA9C20",
        0.3,
        "#CA8323",
        0.35,
        "#B86B25",
        0.4,
        "#A25626",
      ],
      "fill-opacity": 0.75,
      "fill-outline-color": "#B7560A",
    };
    let ownerocc_paint;
    $: ownerocc_paint = {
      "fill-color": [
        "interpolate",
        ["linear"],
        ["get", "Y" + timeFilter],
        0.1,
        "#b490f5",
        0.15,
        "#a279ed",
        0.2,
        "#9a6bf2",
        0.25,
        "#9367e6",
        0.3,
        "#8b63d6",
        0.35,
        "#653ab5",
        0.4,
        "#5c1cd4",
      ],
      "fill-opacity": 0.75,
      "fill-outline-color": "#B7560A",
    };
    mapboxgl.accessToken =
      "pk.eyJ1Ijoib2F2ZWxpbm8iLCJhIjoiY2x2ZngybHdkMGI0NzJra2hxajU0dzF4aSJ9.aL5JC8YN3qfUO2SvBRRLwg";
    onMount(async () => {
      map = new mapboxgl.Map({
        container: "map",
        center: [-71.09415, 42.36027],
        style: "mapbox://styles/mapbox/light-v11",
        zoom: 12,
      });
  
      await new Promise((resolve) => map.on("load", resolve));
      
      map.addSource("corporate_ownership", {
        type: "geojson",
        data: "data/CorpOwner.geojson",
      });
      map.addSource("owner_occupancy", {
        type: "geojson",
        data: "data/OwnerOccupancy.geojson",
      });
  
      updateMapLayer(timeFilter, "corporate_ownership", corpowner_paint);
      updateMapLayer(timeFilter, "owner_occupancy", ownerocc_paint);
      // map.setLayoutProperty("owner_occupancy", "visibility", "none");
      mounted = true;
    });
  
    $: {
      if (mounted) {
        if (toggle == "Corporate Ownership Rates") {
          if (
            map.getLayoutProperty("owner_occupancy", "visibility") == "visible"
          ) {
            map.setLayoutProperty("owner_occupancy", "visibility", "none");
          }
          if (
            map.getLayoutProperty("corporate_ownership", "visibility") == "none"
          ) {
            map.setLayoutProperty("corporate_ownership", "visibility", "visible");
          }
          updateMapLayer(timeFilter, "corporate_ownership", corpowner_paint);
        } else {
          if (map.getLayoutProperty("owner_occupancy", "visibility") == "none") {
            map.setLayoutProperty("owner_occupancy", "visibility", "visible");
          }
          if (
            map.getLayoutProperty("corporate_ownership", "visibility") ==
            "visible"
          ) {
            map.setLayoutProperty("corporate_ownership", "visibility", "none");
          }
          updateMapLayer(timeFilter, "owner_occupancy", ownerocc_paint);
        }
      }
    }
  
    function updateMapLayer(timeFilter, mapID, map_ID_paint) {
      if (!map) return;
      if (map.getLayer(mapID)) {
        map.removeLayer(mapID);
      }
      mapAddLayer(timeFilter, mapID, map_ID_paint);
    }
    function mapAddLayer(timeFilter, mapID, map_ID_paint) {
      map.addLayer({
        id: mapID,
        type: "fill",
        source: mapID,
        layout: {},
        paint: map_ID_paint,
      });
      map.on("click", "corporate_ownership", (e) => {
        map.getCanvas().style.cursor = "pointer";
        const allCoordinates = e.features[0].geometry.coordinates[0];
        const description = e.features[0].properties.Neighborhood;
        // TODO: Want to separate owner occ and corp owner tool tips in the future
        // Right now, just add description on CorpOWner.geojson
        // TODO: Add additional information to description in features, properties dictionary in json file and append/format here
        let sumLong = 0;
        let sumLat = 0;
        for (const coord of allCoordinates) {
          console.log(coord);
          sumLong += coord[0];
          sumLat += coord[1];
        }
        const coordinates = [
          sumLong / allCoordinates.length,
          sumLat / allCoordinates.length,
        ];
        popup.setLngLat(coordinates).setHTML(description).addTo(map);
      });
      // map.on("click", "corporate_ownership", () => {
      //   map.getCanvas().style.cursor = "";
      //   popup.remove();
      // });
    }
    function getCoords(station) {
      let point = new mapboxgl.LngLat(+station.Lon, +station.Lat);
      let { x, y } = map.project(point);
      return { cx: x, cy: y };
    }
  
    let popup = new mapboxgl.Popup({
      closeButton: true,
      closeOnClick: true,
    });
    $: console.log(toggle);

  </script>
  
  <!-- <body> -->
  <header>
    <h1> EvictionWatch</h1>
    <h3> Alejandro Tañón Díaz, Olivia Avelino, Sabrina Su, Sophia Green</h3>
  </header>
  <div class="info">
    <h2>Background Information</h2>
    <p>Over the past few decades, the steady increase in corporate ownership rates in Boston has played a significant role in shaping the city’s housing landscape. 
      This trend has not only impacted owner-occupancy but has also been a key factor in the rising eviction rates, contributing to the ongoing housing crisis.</p>
    <p>In 2020, the Governor of Massachusetts declared a state of emergency due to the COVID-19 pandemic. Shortly after that, on April 20th, 2020, the Governor signed 
      into law “An Act Providing for a Moratorium on Evictions and Foreclosures During the COVID-19 Emergency.” This legislation, a lifeline for many, temporarily 
      halted non-essential evictions and foreclosures in the state until it expired in October 2020. Fortunately, various federal eviction moratoriums were passed during 
      that time. These continued to be extended until August 21st, 2021 when the Supreme Court struck down the Biden Administration’s moratorium.</p>
    <p>This narrative visualization explores how corporate ownership rates affected owner occupancy and evictions in Boston concerning neighborhoods and racial demographics 
      after the COVID-19 emergency eviction moratorium.</p>
  </div>
  <div class="context">
    <h2>Context</h2>
    <div class='graph'>
      <h3>Change in Corporate Ownership Rates per Neighborhood from 2004-2023</h3>
      <Bargraphs_corp/>
    </div>
      <p>Since 2004, corporate ownership rates in Boston have skyrocketed. In every single neighborhood, there was at least a 10% increase in corporate ownership rate. 
        Some neighborhoods, like the Fenway and the South Boston Waterfront, experienced over a 25% increase. </p>
    <div class='graph'>
      <h3>Change in Owner Occupancy Rates per Neighborhood from 2004-2023</h3>
      <Bargraphs_own/>
    </div>
        <p>In this same timeframe, the majority of Boston neighborhoods have experienced a decrease in owner-occupancy rates. The impact of this, in combination with the rise in 
          corporate ownership rates, is evident when we take a look at how eviction rates have changed in these neighborhoods.  </p>
        <figure>
          <img src="image/Graph_6.png" alt="" />
        </figure>
        <p>Eviction rates in Dorchester and Roxbury, both of which have high percentages of Black and Hispanic residents, have increased sharply since the Covid-19 pandemic. 
          We must keep in mind that each of these neighborhoods had nearly a 20% increase in corporate ownership rates since 2004. </p>
  </div>
    <h2>A Closer Look at Corportate Ownership and Owner Occupancy Rates</h2>
    <label>
      Filter by year:
      <input type="range" min="2004" max="2024" bind:value={timeFilter} />
      <time>{timeFilter}</time>
    </label>
    <!-- Corporate Ownership Rates
    <label class="switch">
      <input type="checkbox" />
      <span class="slider round"></span>
    </label> -->
    <Switch
      bind:value={toggle}
      label=""
      design="multi"
      options={["Owner Occupancy Rates", "Corporate Ownership Rates"]}
      fontSize={16}
    />
    <div id="map-container">
  
      <div id="map"></div>
    
    </div>
  
  <style>
    @import url("$lib/global.css");
    img {
          max-width: 80%;
      }
    .context {
      display: grid;
      grid-template-columns: 4fr 1fr;
      
      .graph{
        grid-column: 1;
        padding-bottom: 5%;
      }
      p {
        grid-column: 2;
        display: grid;
        align-items: center
      }
    }
  </style>
  