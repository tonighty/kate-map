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

                    <p>
                        <span class="uk-text-muted">Адрес:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.address }}</span>
                    </p>
                    <p>
                        <span class="uk-text-muted">Сайт:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.site }}</span>
                    </p>
                    <p>
                        <span class="uk-text-muted">Время работы:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis">{{ currentModalData.workTime }}</span>
                    </p>
                    <p>
                        <span class="uk-text-muted">Телефоны:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis" v-for="phone in currentModalData.phones">{{ phone }} <br></span>
                    </p>
                    <p>
                        <span class="uk-text-muted">Можно приобрести экскурсии на следующие предприятия:</span>
                        <br>
                        <span class="uk-text-bold uk-text-emphasis" v-for="tour in currentModalData.availableTours">— {{ tour }} <br></span>
                    </p>

                    <div uk-slideshow>
                        <ul class="uk-slideshow-items">
                            <li>
                                <img src="../assets/factory-marker.png" alt="" uk-cover>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
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

const tourAgencies = [
    {
        name: 'Афина Паллада',
        address: 'ул. Коммунаров, 21, Владивосток (этаж 3, комната 310).',
        site: 'afina-pallada.org',
        workTime: 'пн-пт 09:00–18:00',
        phones: [
            '+7 (423) 257-55-63',
            '+7 (423) 225-24-76',
        ],
        availableTours: [
            'ООО «Фабрика мороженного»',
            'Завод «Соллерс Мазда».',
        ],
        lat: 700,
        lon: 969,
    },
    {
        name: 'другое агентство',
        address: 'ул. Коммунаров, 21, Владивосток (этаж 3, комната 310).',
        description: 'описание',
        email: 'mjr@feip.co',
        instagram: 'someinsta',
        site: 'afina-pallada.org',
        workTime: 'пн-пт 09:00–18:00',
        phones: [
            '+7 (423) 257-55-63',
            '+7 (423) 225-24-76',
        ],
        availableTours: [
            'ООО «Фабрика мороженного»',
            'Завод «Соллерс Мазда».',
        ],
        lat: 400,
        lon: 769,
    },
];

const factories = [
    {
        name: 'Эвернит',
        address: 'ул. Русская, д. 94А, пом. здание Завода «Варяг», г. Владивосток',
        description: 'описание',
        email: 'everneat@everneat.ru',
        instagram: 'www.instagram.com/ever_neat/',
        site: 'https://everneat.ru',
        workTime: 'пн-пт 09:00–18:00',
        phones: [
            '+7 (423) 202-52-53',
            'WA +7 (902)-488-80-70',
        ],
        availableTours: [
            'ООО «Фабрика мороженного»',
            'Завод «Соллерс Мазда».',
        ],
        lat: 1159.5733503104373,
        lon: 845.3045969605365,
    },
];

export default {
    name: "Map",
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

            map.on('click', function(e) {
                alert(`lat: ${Math.round(e.latlng.lat)}, lon: ${Math.round(e.latlng.lng)}`)
            });

            const bounds = [[0, 0], [2663, 1900]]

            L.imageOverlay(mapImage, bounds).addTo(map)

            map.fitBounds(bounds)
            map.setMaxBounds(bounds)
            map.setView([1500, 1500], 0.2)

            const factoryMarker = L.icon({
                iconUrl: factoryMarkerImage,
                shadowUrl: markerShadowImage,
                iconSize: [138 / 2, 176 / 2], // size of the icon
                shadowSize: [138, 176],
                shadowAnchor: [45, 132],
            })

            const tourMarker = L.icon({
                iconUrl: tourMarkerImage,
                shadowUrl: markerShadowImage,
                iconSize: [138 / 2, 176 / 2], // size of the icon
                shadowSize: [138, 176],
                shadowAnchor: [45, 132],
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
