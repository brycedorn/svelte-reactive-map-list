<style>
  #map {
    height: 100vh;
  }

  #map:before {
    box-shadow: 20px 0 10px -10px rgba(0, 0, 0, 0.15) inset;
    content: '';
    height: 100vh;
    left: 0;
    position: absolute;
    width: 20px;
    z-index: 1000;
  }
</style>

<script>
  import mapboxgl from 'mapbox-gl/dist/mapbox-gl.js';
  import { activeListItem, activeMapItem } from './stores.js';
  import { onMount, onDestroy } from 'svelte';
  import { accessToken, listItems } from './consts';

  let mapRef;

  function generateFeature({ name, address, coordinates, website }, index) {
    return {
      type: 'Feature',
      properties: {
        description: `<p><b>${name}</b><br>${address}<br><a href="${website}">${website}</a></p>`,
        id: index
      },
      geometry: {
        type: 'Point',
        coordinates
      }
    };
  }

  onMount(async () => {
    mapboxgl.accessToken = accessToken;

    // Create the map
    mapRef = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: listItems[0].coordinates,
      zoom: 13.5
    });

    mapRef.on('load', function() {
      // Add markers to map
      mapRef.addLayer({
        id: 'places',
        type: 'symbol',
        source: {
          type: 'geojson',
          data: {
            type: 'FeatureCollection',
            features: listItems.map(generateFeature)
          }
        },
        layout: {
          'icon-image': 'cafe',
          'icon-size': 1.5,
          'icon-allow-overlap': true
        }
      });

      // When clicking on a map marker
      mapRef.on('click', 'places', function({ features }) {
        const match = features[0];
        const coordinates = match.geometry.coordinates.slice();

        // Show popup
        new mapboxgl.Popup()
          .setLngLat(coordinates)
          .setHTML(match.properties.description)
          .addTo(mapRef);

        // Set new active list item
        activeListItem.set(match.properties.id);
      });

      // Change the cursor to a pointer when the mouse is over the places layer.
      mapRef.on('mouseenter', 'places', function() {
        mapRef.getCanvas().style.cursor = 'pointer';
      });

      // Change it back to a pointer when it leaves.
      mapRef.on('mouseleave', 'places', function() {
        mapRef.getCanvas().style.cursor = '';
      });
    });
  });

  // Update map center when active list item is updated via list
  const unsubscribeActiveMapItem = activeMapItem.subscribe(newActiveMapItem => {
    if (mapRef) {
      mapRef.flyTo({
        center: listItems[newActiveMapItem].coordinates
      });
    }
  });

  // Remove listener on unmount
  onDestroy(unsubscribeActiveMapItem);
</script>

<div id="map"></div>
