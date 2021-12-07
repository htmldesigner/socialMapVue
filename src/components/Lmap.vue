<template>
  <div class="rel">
    <Info class="info" :selected="selectedPeople"></Info>
    <div
      id="mapid"
      ref="map"
      style="width: 100%; height: 100%; overflow: hidden; position: absolute"
    ></div>
  </div>
</template>

<script>
import Info from "./Info";
// import resultInvalid from "../data/resultinvalid.json";
// import resultLostParent from "../data/resultlostParent.json";
// import resultManyChildren from "../data/resultmanyChildren.json";
// import resultOlder from "../data/resultOlder.json";
import allPiople from "../data/allPiople.json";

export default {
  components: {
    Info,
  },
  name: "Lmap",
  data() {
    return {
      selectedPeople: [],
      // peopleMarkers: [],
      invalidClusterGroup: null,
      olderClusterGroup: null,
      manyChildrenClusterGroup: null,
      lostParentClusterGroup: null,

      invalid: [],
      older: [],
      manyChildren: [],
      lostParent: [],
    };
  },

  methods: {
    mapInstance() {
      const map = L.map(this.$refs.map, { preferCanvas: true }).setView(
        [51.2189539, 51.3726394],
        10
      );

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(map);

      L.control.scale().addTo(map);

      this.invalidClusterGroup = new L.markerClusterGroup({
        maxClusterRadius: 40,
      });
      let greenIcon = new L.Icon({
        iconUrl:
          "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png",
        shadowUrl:
          "https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png",
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });
      for (let unit of allPiople.filter((el) => {
        return el.type === "invalid";
      })) {
        const marker = L.marker(unit.geom.coordinates, { icon: greenIcon });
        marker.unitType = "invalid";
        marker
          .bindPopup(`${unit.address}, ${unit.geom.coordinates}`)
          .openPopup();
        this.invalidClusterGroup.addLayer(marker);
      }

      // insert into invalids (`geometry`, `location`, `geom`, `address`, `formatted_address`, `status`) SELECT `geometry`, `location`, `geom`, `address`, `formatted_address`, `status` from lostparents
      this.olderClusterGroup = new L.markerClusterGroup();
      for (let unit of allPiople.filter((el) => {
        return el.type === "older";
      })) {
        const marker = L.marker(unit.geom.coordinates);
        marker.unitType = "older";
        marker
          .bindPopup(`${unit.address}, ${unit.geom.coordinates}`)
          .openPopup();
        this.olderClusterGroup.addLayer(marker);
      }

      this.manyChildrenClusterGroup = new L.markerClusterGroup({
        maxClusterRadius: 40,
      });
      let orangeIcon = new L.Icon({
        iconUrl:
          "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-orange.png",
        shadowUrl:
          "https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png",
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });
      for (let unit of allPiople.filter((el) => {
        return el.type === "manychildren";
      })) {
        const marker = L.marker(unit.geom.coordinates, { icon: orangeIcon });
        marker.unitType = "manychildren";
        marker
          .bindPopup(`${unit.address}, ${unit.geom.coordinates}`)
          .openPopup();
        this.manyChildrenClusterGroup.addLayer(marker);
      }

      this.lostParentClusterGroup = new L.markerClusterGroup({
        maxClusterRadius: 40,
      });
      let redIcon = new L.Icon({
        iconUrl:
          "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png",
        shadowUrl:
          "https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png",
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      });
      for (let unit of allPiople.filter((el) => {
        return el.type === "lostparent";
      })) {
        const marker = L.marker(unit.geom.coordinates, { icon: redIcon });
        marker.unitType = "lostparent";
        marker
          .bindPopup(`${unit.address}, ${unit.geom.coordinates}`)
          .openPopup();
        this.lostParentClusterGroup.addLayer(marker);
      }

      const layers = {
        "Инвалиды:": this.invalidClusterGroup,
        "Пенсионеры:": this.olderClusterGroup,
        "Многодетные:": this.manyChildrenClusterGroup,
        "Потеря кормильца:": this.lostParentClusterGroup,
      };

      L.featureGroup([
        this.invalidClusterGroup,
        this.olderClusterGroup,
        this.manyChildrenClusterGroup,
        this.lostParentClusterGroup,
      ]).addTo(map);

      L.control.layers(null, layers).addTo(map);

      L.control.lasso({ position: "topleft" }).addTo(map);
      map.on("lasso.finished", this._onLassoFinished, this);
    },

    _onLassoFinished(event) {
      this.selectedPeople = [];
      this._addSelectedMarkers(event.layers, this.selectedPeople);
    },

    _addSelectedMarkers(layers, selectedMarkers) {
      for (const layer of layers) {
        if (layer instanceof L.Marker) {
          selectedMarkers.push(layer);
        } else if (layer instanceof L.MarkerCluster) {
          this._addSelectedMarkers(layer.getAllChildMarkers(), selectedMarkers);
        }
      }
      console.log(this.selectedPeople);
    },
  },
  created() {},
  mounted() {
    this.mapInstance();
  },
};
</script>

<style>
.info {
  position: absolute;
  top: 120px;
  right: 20px;
  z-index: 9999;
  padding: 6px 8px;
  width: 200px;
  font: 14px/16px Arial, Helvetica, sans-serif;
  background: white;
  background: rgba(255, 255, 255, 0.8);
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
}
</style>