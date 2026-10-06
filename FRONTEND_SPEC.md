# Spatial Registry UI - Engineering Spec

## 1. Objective
Build a single-page Vue 3 (Composition API) application to visualize geospatial assets fetched from a Spring Boot REST API.

## 2. Technical Stack
- Vue 3 (Script Setup / Composition API)
- TypeScript
- Leaflet.js (for rendering the map)
- CSS: Standard scoped CSS (no Tailwind needed for this prototype)

## 3. Data Integration
- **Endpoint:** GET http://localhost:8080/api/v1/assets
- **Asset Interface:**
  ```typescript
  interface SpatialAsset {
    id: number;
    assetId: string;
    sensorType: string;
    latitude: number;
    longitude: number;
    cloudCoverPercentage: number;
    capturedAt: string;
  }
  ```

## 4. UI Requirements
1. A full-screen or prominent Leaflet map centered on the US (Lat: 39.0, Lon: -98.0, Zoom: 4).
2. An `onMounted` lifecycle hook that fetches the assets from the API.
3. Iterate through the fetched assets and plot a Leaflet Marker for each asset's `latitude` and `longitude`.
4. Bind a Leaflet Popup to each marker displaying the `assetId` and `sensorType`.