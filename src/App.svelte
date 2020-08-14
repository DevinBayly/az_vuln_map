<script>
  import * as d3 from "d3";
  import Holder from "./Holder.svelte";
  import CB from "./checkBox.svelte";
  import { onMount } from "svelte";
  import { activeProp } from "./store.js";
  onMount(() => {
    let map = L.map("mapcontainer", {
      center: [32.25346, -110.911789], // latitude, longitude in decimal degrees (find it on Google Maps with a right click!)
      zoom: 10, // can be 0-22, higher is closer
      scrollWheelZoom: true // don't zoom the map on scroll
    });
    // add the basemap tiles
    L.tileLayer(
      "https://stamen-tiles.a.ssl.fastly.net/toner-lite/{z}/{x}/{y}@2x.png" // stamen toner tiles
      // stamen toner tiles
    ).addTo(map);

    // get the geojsons
    let mapData = async () => {
      let zctaIp = await fetch("./ZCTA_IP.geojson").then(res => res.json());
      // add properties to check boxes
      let propertyNames = Object.keys(zctaIp.features[0].properties);
      propertyNames.map(
        name =>
          new CB({
            target: document.querySelector("#holder"),
            props: {
              propName:name
            }
          })
      );
      let zctaEd = await fetch("./ZCTA_ED.geojson").then(res => res.json());
      // create options for updating the visualization
      let colscale = d3
        .scaleLinear()
        .domain(d3.extent(zctaIp.features.map(e => e.properties.ZCTA_IP_Ot)))
        .range(["red", "blue"]);
      let ipLayer = L.geoJSON(zctaIp, {
        style: {
          color: "#ff7800",
          weight: 1,
          opacity: 0.95
        }
      }).addTo(map);
      // make a listener for active prop change
      // fli stands for featureinstancelayer, basically its each feature o teh layer
      activeProp.subscribe(prop => {
        if (prop != null) {
          ipLayer.eachLayer(fli => {
            let propValue = fli.feature.properties[prop];
            fli.setStyle({
              fillColor: colscale(propValue)
            });
          });
        }
      });

      let edLayer = L.geoJSON(zctaEd, {
        style: {
          color: "#ff5020",
          weight: 2,
          opacity: 0.65
        }
      });

      var layerControl = new L.Control.Layers(null, {
        ZCTA_IP: ipLayer,
        ZCTA_ED: edLayer
      }).addTo(map);
    };
    mapData();
  });
</script>

<style>
  #mapcontainer {
    height: 100%;
    width: 100%;
  }
</style>

<div id="mapcontainer" />

<Holder />
