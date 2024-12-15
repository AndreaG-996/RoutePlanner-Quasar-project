<template>
  <div class="route-planner">
    <div class="sidebar">
      <h6>Enter Addresses ({{ addresses.length }}/10)</h6>
      <div class="address-note">
        <span class="icon">💡</span>
        <span class="note-text">Please enter your first address as your final destination.</span>
      </div>
      <div v-for="(address, index) in addresses" :key="index">
        <q-input
          v-model="addresses[index]"
          outlined
          dense
          placeholder="Enter address"
          class="address-input"
        />
      </div>
      <q-btn
        label="Add Address"
        color="primary"
        @click="addAddress"
        class="add-btn"
        :disable="addresses.length >= 10"
      />
      <q-btn
        label="Optimise Route"
        color="secondary"
        @click="fetchAndDisplayRoute"
        class="optimise-btn"
      />
    </div>
    <div id="map" class="map-container"></div>
  </div>
</template>

<script>
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import axios from 'axios'

export default {
  name: 'RoutePlanner',
  data() {
    return {
      map: null,
      addresses: [''],
      routeLayer: null,
      markers: [],
    }
  },
  mounted() {
    this.initMap()
  },
  methods: {
    initMap() {
      this.map = L.map('map').setView([51.505, -0.09], 13)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors',
        maxZoom: 19,
      }).addTo(this.map)
    },
    addAddress() {
      if (this.addresses.length < 10) {
        this.addresses.push('')
      } else {
        alert('You can only add up to 10 addresses.')
      }
    },
    async fetchAndDisplayRoute() {
      try {
        const response = await axios.post('http://127.0.0.1:5000/optimise-route', {
          addresses: this.addresses.filter((address) => address.trim() !== ''),
        })
        const routeData = response.data.coordinates
        const optimisedAddresses = response.data.optimised_addresses
        if (!routeData || routeData.length === 0) {
          alert('No route data received!')
          return
        }
        this.drawRoute(routeData, optimisedAddresses)
      } catch (error) {
        console.error('Error fetching route:', error)
        alert('Failed to fetch the optimal route. Please try again.')
      }
    },
    drawRoute(routeData, addresses) {
      console.log('Route Data:', routeData)
      console.log('Addresses:', addresses)
      // Clear previous records first to avoid overlap
      if (this.routeLayer && this.map.hasLayer(this.routeLayer)) {
        this.map.removeLayer(this.routeLayer)
      }
      if (this.markers) {
        this.markers.forEach((marker) => this.map.removeLayer(marker))
      }
      this.markers = []

      // Get coords and draw the path
      const latLngs = routeData.map((point) => [point[0], point[1]])
      console.log('LatLngs:', latLngs)
      this.routeLayer = L.polyline(latLngs, { color: 'info', weight: 5 }).addTo(this.map)
      this.map.fitBounds(this.routeLayer.getBounds())

      //Add marker at each point, with order and address
      latLngs.forEach((coords, index) => {
        const marker = L.marker(coords).addTo(this.map)
        // Add Popup to show order and address
        const popupContent = `<strong>${index + 1}. ${addresses[index]}</strong>`
        marker.bindPopup(popupContent).openPopup()
        // Add Tooltip to show order
        marker.bindTooltip(`${index + 1}`, { permanent: true, className: 'marker-tooltip' })
        // Save markers, for clearing later
        this.markers.push(marker)
      })
    },
  },
}
</script>

<style scoped>
.route-planner {
  display: flex;
  height: 100vh;
}

.sidebar {
  width: 300px;
  background: #b2d3ba;
  padding: 20px;
  overflow: auto;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.sidebar h6 {
  color: #285a3a;
  font-family: 'Comic Sans MS', sans-serif;
  margin-bottom: 20px;
  letter-spacing: 1px;
  transition: color 0.3s ease;
}

.sidebar h6:hover {
  color: #0f3014;
  cursor: grab;
}

.address-note {
  display: flex;
  align-items: center;
  color: #356b49;
  font-size: 0.9em;
  font-family: 'Comic Sans MS', sans-serif;
}

.address-note .icon {
  margin-right: 8px;
  font-size: 1.2em;
  color: #ffcc00;
}

.add-btn {
  font-family: 'Comic Sans MS', sans-serif;
}

.optimise-btn {
  margin-top: auto; /* Put the button at the bottom */
  align-self: flex-end; /* Align to the right */
  padding: 10px 20px;
  font-family: 'Comic Sans MS', sans-serif;
  border-radius: 5px;
}

.map-container {
  flex-grow: 1;
}
</style>
