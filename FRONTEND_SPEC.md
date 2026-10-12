# Spatial Registry UI - Engineering Spec

## 1. Objective
Build a single-page Vue 3 (Composition API) application to visualize geospatial assets fetched from a Spring Boot REST API, and automate deployment to AWS S3/CloudFront.

## 2. Technical Stack
- Vue 3 (Script Setup / Composition API)
- TypeScript
- Vite
- Leaflet.js (for rendering the map)
- CSS: Standard scoped CSS (no Tailwind needed for this prototype)

## 3. Data Integration
- **Endpoint:** GET `{import.meta.env.VITE_API_BASE_URL}/api/v1/assets`
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

## 5. CI/CD Deployment Requirements
Implement a GitHub Actions workflow (`.github/workflows/deploy.yml`) to build and deploy the SPA to AWS.
- **Trigger:** Push to the `main` branch.
- **Security:** Use AWS OpenID Connect (OIDC). Require `id-token: write` and `contents: read` permissions.
- **AWS Region:** `us-east-1`
- **Role ARN:** Sourced from `secrets.AWS_ROLE_ARN`
- **Build Steps:** Setup Node.js, `npm install`, inject `.env` variables, and `npm run build`.
- **S3 Sync:** Sync the Vite `dist` output folder to the exact S3 bucket: `s3://spatial-registry-ui-623045221655` (delete removed files).
- **CloudFront Invalidation:** Invalidate the CloudFront cache path `/*` using the distribution ID sourced from `secrets.CLOUDFRONT_DISTRIBUTION_ID`.