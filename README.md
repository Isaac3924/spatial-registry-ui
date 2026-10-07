# Spatial Registry UI

This is the Vue 3 frontend for the Spatial Registry application, demonstrating an AI-Assisted (Agentic) SDLC approach to building geospatial web interfaces.

## Architecture
- **Framework:** Vue 3 (Composition API) with TypeScript
- **Mapping:** Leaflet.js
- **Build Tool:** Vite
- **Backend Integration:** Designed to consume the [Spatial Registry Boot API](https://github.com/Isaac3924/spatial-asset-registry).

## Engineering Process
This interface was generated using Spec-Driven Development. The core functionality and data contracts were defined in [`FRONTEND_SPEC.md`](./FRONTEND_SPEC.md) before utilizing AI coding agents to implement the UI logic, ensuring strict adherence to the required data models and component structure.

## Local Development
1. Ensure the Java Spring Boot backend is running on port 8080.
2. Install dependencies: `npm install`
3. Run the development server `npm run dev`