<template>
    <div id="container">
        <div id="mapContainer"></div>
    </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import mapImage from "../assets/map.png";

export default {
    name: "Map",
    data() {
        return {
            center: [37, 7749, -122, 4194]
        }
    },
    methods: {
        setupLeafletMap: function () {
            const map = L.map("mapContainer", {
                crs: L.CRS.Simple,
                minZoom: 0,
                zoomSnap: 0,
                zoomDelta: 1,
                dragging: !L.Browser.mobile,
                tap: !L.Browser.mobile,
            })

            const bounds = [[0, 0], [2663, 1900]]

            L.imageOverlay(mapImage, bounds).addTo(map)

            map.fitBounds(bounds)
            map.setMaxBounds(bounds)
            map.setView([1500, 1500], 0.2)

            const golum = L.icon({
                iconUrl: 'golum.png',
                iconSize: [100, 100], // size of the icon
            })

            L.marker([1500, 1500], {icon: golum}).addTo(map)
        },
    },
    mounted() {
        this.setupLeafletMap();
    },
};
</script>

<style scoped>
#mapContainer {
    width: 100vw;
    height: 100vh;
}
</style>
