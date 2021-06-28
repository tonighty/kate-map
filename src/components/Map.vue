<template>
    <div id="container">
        <div id="mapContainer"></div>

        <a href="#" class="marker-toggle marker-toggle__agency" :class="agencyEnabled ? '' : 'marker-toggle_disabled'" v-on:click="toggleAgencies">
            <img src="../assets/tour-marker.png" alt="Вкл/выкл">
        </a>

        <a href="#" class="marker-toggle marker-toggle__factory" :class="factoryEnabled ? '' : 'marker-toggle_disabled'" v-on:click="toggleFactories">
            <img src="../assets/factory-marker.png" alt="Вкл/выкл">
        </a>

        <div id="marker-info-modal" uk-modal>
            <div class="uk-modal-dialog uk-border-rounded">
                <button class="uk-modal-close-default" type="button" uk-close></button>
                <div class="uk-modal-body">
                    <h2 class="uk-modal-title uk-text-bold uk-text-center">{{ currentModalData.name }}</h2>

                    <nl2br tag="p" v-if="currentModalData.description" :text="currentModalData.description">
                        {{ currentModalData.description }}
                    </nl2br>
                    <p>
                        <span class="uk-text-muted">Адрес:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.address }}</span>
                    </p>
                    <p v-if="currentModalData.site">
                        <span class="uk-text-muted">Сайт:</span>
                        <br>
                        <a :href="currentModalData.site" class="uk-link">{{ currentModalData.site }}</a>
                    </p>
                    <p v-if="currentModalData.email">
                        <span class="uk-text-muted">E-mail:</span>
                        <br>
                        <a :href="'mailto:' + currentModalData.email" class="uk-link">{{ currentModalData.email }}</a>
                    </p>
                    <p v-if="currentModalData.instagram">
                        <span class="uk-text-muted">Instagram:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.instagram }}</span>
                    </p>
                    <p v-if="currentModalData.workTime">
                        <span class="uk-text-muted">Время работы:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.workTime }}</span>
                    </p>
                    <p>
                        <span class="uk-text-muted">Телефоны:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis" v-for="phone in currentModalData.phones">{{ phone }} <br></span>
                    </p>
                    <p v-if="currentModalData.availableTours">
                        <span class="uk-text-muted">Можно приобрести экскурсии на следующие предприятия:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis" v-for="tour in currentModalData.availableTours">— {{ tour }} <br></span>
                    </p>

                    <div uk-slideshow="autoplay: true" v-if="currentModalData.pic">
                        <div class="uk-position-relative uk-visible-toggle uk-light" tabindex="-1">
                            <ul class="uk-slideshow-items">
                                <li v-for="image in currentModalData.pic">
                                    <img :src="image" alt="No alt" uk-cover>
                                </li>
                            </ul>

                            <a class="uk-position-center-left uk-position-small uk-hidden-hover" href="#" uk-slidenav-previous uk-slideshow-item="previous"></a>
                            <a class="uk-position-center-right uk-position-small uk-hidden-hover" href="#" uk-slidenav-next uk-slideshow-item="next"></a>
                        </div>
                        <ul class="uk-slideshow-nav uk-dotnav uk-flex-center uk-margin"></ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Nl2br from "vue-nl2br";
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import UIkit from "uikit";
import "leaflet.animatedmarker/src/AnimatedMarker";
import mapImage from "../assets/map.png";
import factoryMarkerImage from "../assets/factory-marker.png";
import tourMarkerImage from "../assets/tour-marker.png";
import markerShadowImage from "leaflet/dist/images/marker-shadow.png";
import planeImage from "../assets/plane.png";
import bigCloudImage from "../assets/cloud-big.png";

/** @type {Array} */
import tourAgencies from "../assets/tour-firms.json";
/** @type {Array} */
import factories from "../assets/fabrics.json";

export default {
    name: "Map",
    components: {
        Nl2br
    },
    data() {
        return {
            center: [37, 7749, -122, 4194],
            currentModalData: tourAgencies[0],
            agencyEnabled: true,
            factoryEnabled: true,
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

            const factoryMarker = L.icon({
                iconUrl: factoryMarkerImage,
                shadowUrl: markerShadowImage,
                iconSize: [138 / 2, 176 / 2], // size of the icon
                iconAnchor: [138 / 4, 176 / 2],
                shadowSize: [138, 176],
                shadowAnchor: [138 / 4 + 10, 176],
            })

            const tourMarker = L.icon({
                iconUrl: tourMarkerImage,
                shadowUrl: markerShadowImage,
                iconSize: [138 / 2, 176 / 2], // size of the icon
                iconAnchor: [138 / 4, 176 / 2],
                shadowSize: [138, 176],
                shadowAnchor: [138 / 4 + 10, 176],
            })

            const modal = UIkit.modal('#marker-info-modal')
            const agencyLayers = [];
            const factoryLayers = [];

            tourAgencies.forEach((item) => {
                agencyLayers.push(L.marker([item.lat, item.lon], {icon: tourMarker}).on('click', () => {
                    this.currentModalData = item;
                    modal.show();
                }));
            });

            factories.forEach((item) => {
                factoryLayers.push(L.marker([item.lat, item.lon], {icon: factoryMarker}).on('click', () => {
                    this.currentModalData = item;
                    modal.show();
                }));
            });

            this.agencyGroup = L.layerGroup(agencyLayers).addTo(map)
            this.factoryGroup = L.layerGroup(factoryLayers).addTo(map)
            this.map = map

            this.addAnimatedMarker(map, {
                dots: [[2000, -400],[2100, 3000]],
                iconUrl: planeImage,
                iconSize: [340 / 2, 268 / 2],
                interval: 5000,
            })

            this.addAnimatedMarker(map, {
                dots: [[500, -300],[700, 3000]],
                iconUrl: planeImage,
                iconSize: [340 / 2, 268 / 2],
                interval: 10000,
            })

            this.addAnimatedMarker(map, {
                dots: [[2200, -300],[2200, 3000]],
                iconUrl: bigCloudImage,
                iconSize: [567, 276],
                interval: 20000,
            })
            this.addAnimatedMarker(map, {
                dots: [[1500, -300],[1500, 3000]],
                iconUrl: bigCloudImage,
                iconSize: [567 / 2, 276 / 2],
                interval: 15000,
            })
        },
        addAnimatedMarker : function(map, options) {
            const line = L.polyline(options.dots);

            const icon = L.icon({
                iconUrl: options.iconUrl,
                iconSize: options.iconSize,
            })

            let animatedMarker;
            const start = () => {
                animatedMarker = L.animatedMarker(line.getLatLngs(), {
                    icon: icon,
                    interval: options.interval,
                });
                map.addLayer(animatedMarker);
            }

            start()
            setInterval(() => {
                map.removeLayer(animatedMarker)
                start()
            }, options.interval * 2)
        },
        toggleAgencies: function() {
            this.agencyEnabled ? this.agencyGroup.remove() : this.agencyGroup.addTo(this.map);
            this.agencyEnabled = !this.agencyEnabled;
        },
        toggleFactories: function() {
            this.factoryEnabled ? this.factoryGroup.remove() : this.factoryGroup.addTo(this.map);
            this.factoryEnabled = !this.factoryEnabled;
        }
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
.marker-toggle {
    position: absolute;
    z-index: 1000;
    left: 10px;
    bottom: 50%;
    width: 50px;
    height: 50px;
    transition: transform 0.2s, filter 0.2s;
}
.marker-toggle:hover {
    transform: scale(1.2);
}
.marker-toggle__factory {
    bottom: calc(50% - 100px);
}
.marker-toggle_disabled {
    filter: grayscale(1);
}
</style>
