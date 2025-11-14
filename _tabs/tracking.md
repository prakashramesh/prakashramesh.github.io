---
layout: page
icon: fas fa-map-marker-alt
order: 5
title: Visit Tracking
description: Track visits to this site from various geolocations around the world
---

<div id="tracking-container">
  <div class="tracking-header">
    <h2>Visit Tracking</h2>
    <p>This page tracks visits to the site from various geolocations. Data is stored using a free online service.</p>
  </div>

  <div class="tracking-stats">
    <div class="stat-card">
      <h3 id="total-visits">0</h3>
      <p>Total Visits</p>
    </div>
    <div class="stat-card">
      <h3 id="unique-locations">0</h3>
      <p>Unique Locations</p>
    </div>
    <div class="stat-card">
      <h3 id="current-location">-</h3>
      <p>Your Location</p>
    </div>
  </div>

  <div id="map-container">
    <h3>Visit Locations Map</h3>
    <div id="visits-map"></div>
  </div>

  <div id="visits-table-container">
    <h3>Visit History</h3>
    <table id="visits-table">
      <thead>
        <tr>
          <th>Timestamp</th>
          <th>Location</th>
          <th>City</th>
          <th>Region</th>
          <th>Country</th>
          <th>IP Address</th>
        </tr>
      </thead>
      <tbody id="visits-tbody">
        <tr>
          <td colspan="6" class="loading">Loading visit data...</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

<style>
  #tracking-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  .tracking-header {
    text-align: center;
    margin-bottom: 30px;
  }

  .tracking-header h2 {
    margin-bottom: 10px;
  }

  .tracking-stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
  }

  .stat-card {
    background: var(--card-bg-color);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 20px;
    text-align: center;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .stat-card h3 {
    font-size: 2em;
    margin: 0 0 10px 0;
    color: var(--link-color);
  }

  .stat-card p {
    margin: 0;
    color: var(--text-muted-color);
  }

  #map-container {
    margin: 30px 0;
  }

  #map-container h3 {
    margin-bottom: 15px;
  }

  #visits-map {
    width: 100%;
    height: 500px;
    border-radius: 8px;
    border: 1px solid var(--border-color);
    background: var(--card-bg-color);
  }

  .custom-marker {
    background: transparent !important;
    border: none !important;
  }

  @media (max-width: 768px) {
    #visits-map {
      height: 350px;
    }
  }

  #visits-table-container {
    margin-top: 30px;
  }

  #visits-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    background: var(--card-bg-color);
    border-radius: 8px;
    overflow: hidden;
  }

  #visits-table thead {
    background: var(--link-color);
    color: white;
  }

  #visits-table th {
    padding: 12px;
    text-align: left;
    font-weight: 600;
  }

  #visits-table td {
    padding: 12px;
    border-bottom: 1px solid var(--border-color);
  }

  #visits-table tbody tr:hover {
    background: var(--hover-color);
  }

  #visits-table tbody tr:last-child td {
    border-bottom: none;
  }

  .loading {
    text-align: center;
    color: var(--text-muted-color);
    font-style: italic;
  }

  @media (max-width: 768px) {
    #visits-table {
      font-size: 12px;
    }

    #visits-table th,
    #visits-table td {
      padding: 8px;
    }

    .tracking-stats {
      grid-template-columns: 1fr;
    }
  }
</style>

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
     integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
     crossorigin=""/>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
     integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
     crossorigin=""></script>

{% raw %}
<script>
  (function() {
    // API endpoints
    const GEOLOCATION_API = 'https://ipapi.co/json/';
    // JSONBin.io - Free JSON storage service
    // Replace 'YOUR_BIN_ID' with your actual bin ID from jsonbin.io
    // To get a bin ID: 1. Go to https://jsonbin.io 2. Create a free account 3. Create a new bin 4. Copy the bin ID
    const JSONBIN_BIN_ID = '6916a610d0ea881f40e7573d'; // TODO: Replace with your actual bin ID
    const JSONBIN_API_URL = 'https://api.jsonbin.io/v3/b/' + JSONBIN_BIN_ID;
    const JSONBIN_API_KEY = ''; // Optional: Add your API key for private bins (get from jsonbin.io)
    
    let map = null;
    let markers = [];
    let visits = [];

    // Initialize map
    function initMap() {
      if (map) {
        return; // Map already initialized
      }

      // Create map centered on world
      map = L.map('visits-map').setView([20, 0], 2);

      // Add OpenStreetMap tiles
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        maxZoom: 19
      }).addTo(map);
    }

    // Update map with visit markers
    function updateMap() {
      // Wait for map container to be available
      const mapContainer = document.getElementById('visits-map');
      if (!mapContainer) {
        return;
      }

      // Initialize map if not already done
      if (!map) {
        initMap();
        // Wait a bit for map to fully initialize
        setTimeout(() => updateMap(), 100);
        return;
      }

      // Clear existing markers
      markers.forEach(marker => {
        if (map.hasLayer(marker)) {
          map.removeLayer(marker);
        }
      });
      markers = [];

      // Filter visits with valid coordinates
      const validVisits = visits.filter(v => {
        const lat = parseFloat(v.latitude);
        const lng = parseFloat(v.longitude);
        return !isNaN(lat) && !isNaN(lng) && lat !== null && lng !== null && 
               lat >= -90 && lat <= 90 && lng >= -180 && lng <= 180;
      });

      if (validVisits.length === 0) {
        // If no valid visits, center on world view
        map.setView([20, 0], 2);
        return;
      }

      // Create bounds for all markers
      const bounds = L.latLngBounds([]);

      validVisits.forEach((visit, index) => {
        const lat = parseFloat(visit.latitude);
        const lng = parseFloat(visit.longitude);

        // Create custom icon with different colors
        const isRecent = index < 5; // Highlight recent 5 visits
        const iconColor = isRecent ? '#ff0000' : '#3388ff';
        
        const customIcon = L.divIcon({
          className: 'custom-marker',
          html: '<div style="background-color: ' + iconColor + '; width: 14px; height: 14px; border-radius: 50%; border: 2px solid white; box-shadow: 0 2px 6px rgba(0,0,0,0.4);"></div>',
          iconSize: [14, 14],
          iconAnchor: [7, 7]
        });

        const marker = L.marker([lat, lng], { icon: customIcon }).addTo(map);
        
        // Add popup with visit info
        const date = new Date(visit.timestamp);
        const formattedDate = date.toLocaleString();
        marker.bindPopup(
          '<div style="min-width: 200px;">' +
            '<strong>' + visit.city + ', ' + visit.country + '</strong><br>' +
            '<small>' + visit.region + '</small><br>' +
            '<small style="color: #666;">' + formattedDate + '</small><br>' +
            '<small style="color: #999;">IP: ' + visit.ip + '</small>' +
          '</div>'
        );

        markers.push(marker);
        bounds.extend([lat, lng]);
      });

      // Fit map to show all markers
      if (bounds.isValid() && bounds.getNorth() !== bounds.getSouth() && bounds.getEast() !== bounds.getWest()) {
        // Multiple distinct points - fit bounds
        map.fitBounds(bounds, { padding: [50, 50], maxZoom: 15 });
      } else if (validVisits.length === 1) {
        // Single point - center on it with reasonable zoom
        const singleVisit = validVisits[0];
        map.setView([parseFloat(singleVisit.latitude), parseFloat(singleVisit.longitude)], 10);
      } else {
        // Fallback: center on world view
        map.setView([20, 0], 2);
      }

      // Force map to invalidate size in case container was hidden
      setTimeout(() => {
        map.invalidateSize();
      }, 100);
    }

    // Fetch visits from JSONBin.io
    async function fetchVisits() {
      try {
        if (!JSONBIN_BIN_ID || JSONBIN_BIN_ID === 'YOUR_BIN_ID') {
          console.warn('JSONBin.io bin ID not configured. Please set JSONBIN_BIN_ID in the code.');
          visits = [];
          return [];
        }

        const headers = {
          'Content-Type': 'application/json'
        };
        
        if (JSONBIN_API_KEY) {
          headers['X-Master-Key'] = JSONBIN_API_KEY;
        }

        const response = await fetch(JSONBIN_API_URL + '/latest', {
          method: 'GET',
          headers: headers
        });

        if (!response.ok) {
          // If bin doesn't exist, return empty array
          if (response.status === 404) {
            visits = [];
            return [];
          }
          throw new Error('Failed to fetch visits');
        }

        const result = await response.json();
        visits = result.record && Array.isArray(result.record) ? result.record : [];
        return visits;
      } catch (error) {
        console.error('Error fetching visits:', error);
        visits = [];
        return [];
      }
    }

    // Save visits to JSONBin.io
    async function saveVisits(visitsArray) {
      try {
        if (!JSONBIN_BIN_ID || JSONBIN_BIN_ID === 'YOUR_BIN_ID') {
          console.warn('JSONBin.io bin ID not configured. Cannot save visits.');
          return false;
        }

        const headers = {
          'Content-Type': 'application/json'
        };
        
        if (JSONBIN_API_KEY) {
          headers['X-Master-Key'] = JSONBIN_API_KEY;
        }

        const response = await fetch(JSONBIN_API_URL, {
          method: 'PUT',
          headers: headers,
          body: JSON.stringify(visitsArray)
        });

        if (!response.ok) {
          throw new Error('Failed to save visits');
        }

        visits = visitsArray;
        return true;
      } catch (error) {
        console.error('Error saving visits:', error);
        throw error;
      }
    }

    // Save a new visit
    async function saveVisit(visitData) {
      try {
        // Fetch current visits
        await fetchVisits();
        
        // Add new visit at the beginning
        visits.unshift(visitData);
        
        // Keep only last 1000 visits to avoid storage issues
        if (visits.length > 1000) {
          visits.splice(1000);
        }
        
        // Save updated visits
        await saveVisits(visits);
        return visitData;
      } catch (error) {
        console.error('Error saving visit:', error);
        throw error;
      }
    }

    // Get current location data
    async function getCurrentLocation() {
      try {
        const response = await fetch(GEOLOCATION_API);
        const data = await response.json();
        return {
          ip: data.ip || 'Unknown',
          city: data.city || 'Unknown',
          region: data.region || 'Unknown',
          country: data.country_name || 'Unknown',
          countryCode: data.country_code || 'Unknown',
          latitude: data.latitude || null,
          longitude: data.longitude || null,
          timezone: data.timezone || 'Unknown',
          timestamp: new Date().toISOString()
        };
      } catch (error) {
        console.error('Error fetching location:', error);
        return {
          ip: 'Unknown',
          city: 'Unknown',
          region: 'Unknown',
          country: 'Unknown',
          countryCode: 'Unknown',
          latitude: null,
          longitude: null,
          timezone: 'Unknown',
          timestamp: new Date().toISOString()
        };
      }
    }

    // Add a new visit
    async function addVisit() {
      const location = await getCurrentLocation();
      await saveVisit(location);
      return location;
    }

    // Display visits
    function displayVisits() {
      const tbody = document.getElementById('visits-tbody');
      
      // Update stats
      document.getElementById('total-visits').textContent = visits.length;
      
      // Count unique locations
      const uniqueLocations = new Set(visits.map(v => v.city + ', ' + v.country));
      document.getElementById('unique-locations').textContent = uniqueLocations.size;
      
      // Show current location
      if (visits.length > 0) {
        const current = visits[0];
        document.getElementById('current-location').textContent = 
          current.city + ', ' + current.country;
      } else {
        document.getElementById('current-location').textContent = '-';
      }
      
      // Update map
      updateMap();
      
      // Clear and populate table
      tbody.innerHTML = '';
      
      if (visits.length === 0) {
        tbody.innerHTML = '<tr><td colspan="6" class="loading">No visits recorded yet.</td></tr>';
        return;
      }
      
      visits.forEach(visit => {
        const row = document.createElement('tr');
        const date = new Date(visit.timestamp);
        const formattedDate = date.toLocaleString();
        
        row.innerHTML = 
          '<td>' + formattedDate + '</td>' +
          '<td>' + visit.city + ', ' + visit.country + '</td>' +
          '<td>' + visit.city + '</td>' +
          '<td>' + visit.region + '</td>' +
          '<td>' + visit.country + ' (' + visit.countryCode + ')</td>' +
          '<td>' + visit.ip + '</td>';
        
        tbody.appendChild(row);
      });
    }

    // Initialize
    async function init() {
      // Check if JSONBin is configured
      if (!JSONBIN_BIN_ID || JSONBIN_BIN_ID === 'YOUR_BIN_ID') {
        const tbody = document.getElementById('visits-tbody');
        if (tbody) {
          tbody.innerHTML = '<tr><td colspan="6" class="loading" style="color: #dc3545;">⚠️ Please configure JSONBin.io bin ID in the code. See instructions in the code comments.</td></tr>';
        }
        return;
      }

      // Fetch existing visits first
      await fetchVisits();
      
      // Display all visits
      displayVisits();
      
      // Add current visit
      try {
        await addVisit();
        await fetchVisits(); // Refresh after adding
        displayVisits();
      } catch (error) {
        console.error('Error adding visit:', error);
        // Still display existing visits even if adding new one fails
      }
    }

    // Run when DOM is ready
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', init);
    } else {
      init();
    }
  })();
</script>
{% endraw %}

