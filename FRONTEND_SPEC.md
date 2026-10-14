# Spatial Registry UI - Engineering Spec

## 1. Objective
Build a single-page Vue 3 (Composition API) application to visualize geospatial assets fetched from a Spring Boot REST API, allow users to create new assets via map interaction, and automate deployment to AWS S3/CloudFront.

## 2. Technical Stack
- Vue 3 (Script Setup / Composition API)
- TypeScript
- Vite
- Leaflet.js (for rendering the map)
- CSS: Standard scoped CSS (no Tailwind needed for this prototype)

## 3. Data Integration
- **Base URL:** `${import.meta.env.VITE_API_BASE_URL}/api/v1/assets`
- **Endpoints:**
  - `GET /` - Fetch all assets
  - `POST /` - Create a new asset
- **Asset Interface:**
  ```typescript
  interface SpatialAsset {
    id?: number; // Optional for creation, returned by backend
    assetId: string;
    sensorType: string;
    latitude: number;
    longitude: number;
    cloudCoverPercentage?: number;
    capturedAt: string; // ISO-8601 format (e.g., "2026-10-14T15:33:00Z")
    imageUrl?: string;
  }
  ```

## 4. UI Requirements
1. **Map View:** A prominent Leaflet map centered on the US (Lat: 39.0, Lon: -98.0, Zoom: 4).
2. **Data Loading:** An `onMounted` lifecycle hook that fetches the assets via GET and plots them as Leaflet Markers with popups displaying the `assetId` and `sensorType`.
3. **Creation Panel:** A collapsible side menu or overlay card containing a form to create a new `SpatialAsset`.
  - Fields: Asset ID (text), Sensor Type (text), Latitude (number), Longitude (number), Cloud Cover % (number), Image URL (text).
  - The `capturedAt` timestamp should be generated automatically by the frontend (`new Date().toISOString()`) upon submission.
4. **Map Interaction:** Clicking anywhere on the Leaflet map should automatically populate the Latitude and Longitude input fields in the creation form.
5. **Submission:** Submitting the form should send a POST request to the API. On success, instantly plot the new marker on the map, clear the form, and show a brief success message.

## 5. CI/CD Deployment Requirements
Implement a GitHub Actions workflow (`.github/workflows/deploy.yml`) to build and deploy the SPA to AWS.
- **Trigger:** Push to the `main` branch.
- **Security:** Use AWS OpenID Connect (OIDC). Require `id-token: write` and `contents: read` permissions.
- **AWS Region:** `us-east-1`
- **Role ARN:** Sourced from `secrets.AWS_ROLE_ARN`
- **Build Steps:** Setup Node.js, `npm install`, inject `.env` variables, and `npm run build`.
- **S3 Sync:** Sync the Vite `dist` output folder to the exact S3 bucket: `s3://spatial-registry-ui-623045221655` (delete removed files).
- **CloudFront Invalidation:** Invalidate the CloudFront cache path `/*` using the distribution ID sourced from `secrets.CLOUDFRONT_DISTRIBUTION_ID`.