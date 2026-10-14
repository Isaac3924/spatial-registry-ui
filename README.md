# Spatial Registry UI

This is the Vue 3 frontend for the Spatial Registry application, demonstrating an AI-Assisted (Agentic) SDLC approach to building geospatial web interfaces.

## Architecture
- **Framework:** Vue 3 (Composition API) with TypeScript
- **Mapping:** Leaflet.js
- **Build Tool:** Vite
- **Hosting:** AWS S3 distributed via Amazon CloudFront
- **CI/CD:** GitHub Actions with passwordless AWS OIDC authentication
- **Backend Integration:** Designed to consume the [Spatial Registry Boot API](https://github.com/Isaac3924/spatial-asset-registry).

## Features
- **Map Visualization:** Renders spatial assets on an interactive Leaflet map.
- **Data Creation:** Includes a collapsible UI panel and map-click integration to capture coordinates and submit new assets to the backend database.

## Engineering Process
This interface and its deployment pipelines were generated using Spec-Driven Development. The core functionality, data contracts, and CI/CD requirements were defined in [`FRONTEND_SPEC.md`](./FRONTEND_SPEC.md) before utilizing AI coding agents to implement the logic, ensuring strict adherence to the required models and infrastructure constraints.

## Local Development
1. Ensure the Java Spring Boot backend is running on port 8080.
2. Create a `.env` file in the project root and define the local API endpoint:
   `VITE_API_BASE_URL=http://localhost:8080`
3. Install dependencies: `npm install`
4. Run the development server: `npm run dev`

## Deployment
This application uses a fully automated CI/CD pipeline. Pushing changes to the `main` branch triggers a GitHub Actions workflow that:
1. Pulls the target backend API endpoint from the `VITE_API_BASE_URL` GitHub Secret.
2. Compiles the Vite production assets.
3. Authenticates securely with AWS using an OIDC IAM role.
4. Syncs the compiled assets to the private AWS S3 hosting bucket.
5. Invalidates the CloudFront distribution cache to serve the latest updates globally.