<template>
    <div id="container">
        <div id="mapContainer"></div>

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
import mapImage from "../assets/map.png";
import factoryMarkerImage from "../assets/factory-marker.png";
import tourMarkerImage from "../assets/tour-marker.png";
import markerShadowImage from "leaflet/dist/images/marker-shadow.png";

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

            tourAgencies.forEach((item) => {
                L.marker([item.lat, item.lon], {icon: tourMarker}).addTo(map).on('click', () => {
                    this.currentModalData = item;
                    modal.show();
                })
            });

            factories.forEach((item) => {
                L.marker([item.lat, item.lon], {icon: factoryMarker}).addTo(map).on('click', () => {
                    this.currentModalData = item;
                    modal.show();
                })
            });
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
