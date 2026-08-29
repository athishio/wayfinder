# WayFinder

WayFinder is an AI-powered spatial intelligence and location assistance web application. It combines camera capture, GPS positioning, reverse geocoding, and multimodal AI analysis to identify surroundings, verify physical location, discover points of interest, and provide interactive navigation routing.

---

## Features

- **Live Camera Capture**: Uses `navigator.mediaDevices.getUserMedia` (requesting environment/rear-facing camera by default) and an HTML5 canvas to capture images directly from the device.
- **GPS Telemetry & Reverse Geocoding**: Reads device GPS coordinates via the Geolocation API (with support for manual coordinate overrides) and resolves them into a physical street address using the OpenStreetMap Nominatim API.
- **Multimodal AI Analysis**: Analyzes captured imagery alongside geocoded location data using Google's Gemini API (`gemini-3.5-flash`).
  - Describes the immediate physical environment and visible objects.
  - Confirms the user's resolved physical address.
  - Suggests 2 nearby points of interest / tourist spots.
- **Embedded Google Maps & Routing**: Displays the current location on an interactive embedded Google Map, with directions and routing capabilities between origin and destination for multiple travel modes (driving, walking, bicycling, transit).
- **Session Telemetry Logs**: Caches recent scan reports in browser `localStorage` with options to view detailed intelligence markdown reports or export logs to CSV.

---

## How It Works

```mermaid
flowchart TD
    %% Styling Configuration
    classDef clientBox fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#f8fafc,rx:10px,ry:10px;
    classDef serverBox fill:#0f172a,stroke:#818cf8,stroke-width:2px,color:#f8fafc,rx:10px,ry:10px;
    classDef aiBox fill:#064e3b,stroke:#34d399,stroke-width:2px,color:#ecfdf5,rx:10px,ry:10px;
    classDef uiBox fill:#1e1b4b,stroke:#a78bfa,stroke-width:2px,color:#f5f3ff,rx:10px,ry:10px;
    classDef processBox fill:#1e293b,stroke:#64748b,stroke-width:1.5px,color:#f1f5f9,rx:8px,ry:8px;

    subgraph CLIENT [" 🖥️ CLIENT TIER (Browser & Device Sensors) "]
        direction TB
        CAM["📷 <b>Device Camera Stream</b><br/>navigator.mediaDevices.getUserMedia (Rear/Environment)"]:::processBox
        CANVAS["🖼️ <b>HTML5 Canvas Buffer</b><br/>Extracts active frame to JPEG Blob / File payload"]:::clientBox
        GPS["📍 <b>Geolocation Telemetry</b><br/>navigator.geolocation.getCurrentPosition (or Manual Override)"]:::processBox
        
        CAM -->|Capture Action| CANVAS
    end

    subgraph BACKEND [" ⚙️ BACKEND TIER (Next.js API Route: /api/analyze) "]
        direction TB
        GEO["🗺️ <b>Nominatim OpenStreetMap Service</b><br/>Reverse-geocodes Latitude & Longitude to physical address"]:::serverBox
        PROMPT["🧩 <b>Prompt & Context Assembler</b><br/>Builds strict spatial verification prompt + base64 image part"]:::processBox
        GEMINI["✨ <b>Google Gemini 3.5 Flash</b><br/>Multimodal Vision Model: Analyzes environment & names POIs"]:::aiBox
        
        GEO -->|Ground-Truth Address| PROMPT
        PROMPT -->|Multimodal Payload| GEMINI
    end

    subgraph PRESENTATION [" 🧭 PRESENTATION & COMMAND CENTER "]
        direction TB
        REPORT["📋 <b>Spatial Intelligence Report</b><br/>• Environment Analysis<br/>• Verified Street Address<br/>• 2 Nearby Tourist Spots / POIs"]:::aiBox
        MAPS["🗺️ <b>Embedded Google Maps Engine</b><br/>• Real-time Spatial Pinning<br/>• Live Turn-by-Turn Routing (Drive / Walk / Transit)"]:::uiBox
        STORAGE["💾 <b>Client Telemetry Database</b><br/>localStorage scan cache, modal reader & CSV export"]:::processBox

        GEMINI -->|ai_text payload| REPORT
        REPORT --> MAPS
        REPORT --> STORAGE
    end

    CANVAS ==>|POST multipart/form-data: campus_image| BACKEND
    GPS ==>|POST multipart/form-data: lat / lng| GEO
    BACKEND ==>|HTTP 200 JSON: ai_text| PRESENTATION
```

1. **Capture**: The user launches the live camera feed (or uploads an image). A frame is captured to a canvas and converted to an image payload.
2. **Geolocate & Geocode**: The client retrieves GPS coordinates. When sent to the backend (`/api/analyze`), coordinates are reverse-geocoded into a human-readable street address using the OpenStreetMap Nominatim service (`https://nominatim.openstreetmap.org/reverse`).
3. **Analyze**: The backend sends the image and resolved address to Google Gemini (`gemini-3.5-flash`), which generates an environment description, confirms the location, and highlights nearby spots.
4. **Navigate**: The AI report is rendered in markdown alongside an embedded Google Maps view allowing the user to plan routes to destinations.

---

## Tech Stack

- **Framework**: [Next.js](https://nextjs.org/) (App Router)
- **UI & Styling**: React 19, [Tailwind CSS](https://tailwindcss.com/), [Lucide React](https://lucide.dev/), `next-themes`
- **AI & Multimodal**: [`@google/generative-ai`](https://www.npmjs.com/package/@google/generative-ai) (`gemini-3.5-flash`)
- **Geocoding**: OpenStreetMap Nominatim API
- **Maps**: Google Maps Embed

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (version 18+ recommended)
- A Google Gemini API Key (obtainable from [Google AI Studio](https://aistudio.google.com/))

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/athishio/wayfinder.git
   cd wayfinder
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure Environment Variables:**
   Create a `.env.local` file in the root directory:
   ```env
   GEMINI_API_KEY=your_gemini_api_key_here
   ```

4. **Run the development server:**
   ```bash
   npm run dev
   ```

5. **Open the application:**
   Navigate to [http://localhost:3000](http://localhost:3000) in your web browser. Ensure camera and location permissions are granted when prompted.

---

## Scripts

- `npm run dev` - Starts the development server.
- `npm run build` - Builds the application for production.
- `npm run start` - Starts the production server.
- `npm run lint` - Runs ESLint code quality checks.
