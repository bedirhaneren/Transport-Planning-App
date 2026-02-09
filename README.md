# Public Transport Route Planning

This project is a **Windows Forms-based public transport route planning application** that visualizes bus and tram stops on an interactive map and calculates optimal routes based on distance, time, and cost. The application uses a **graph-based shortest path algorithm** over a predefined transport network loaded from a JSON dataset.

The UI is built on **GMap.NET**, allowing the user to see stops, planned routes, and a direct taxi route on a real map.

---

## Features

- **Interactive map (GMap.NET)**  
  - Displays bus and tram stops on a Google Map.
  - Shows two different public transport routes and one direct taxi route.

- **Shortest path calculation**
  - Graph 1: edges weighted by **fare (ucret)**.
  - Graph 2: edges weighted by **travel time (sure)**.
  - Uses a Dijkstra-like algorithm to compute the shortest path between stops.

- **Nearest stop detection**
  - Given:
    - Current coordinates (`startLat`, `startLon`)
    - Target coordinates (`targetLat`, `targetLon`)
  - Finds:
    - Closest start stop to the user
    - Closest target stop to the destination

- **Taxi fare calculation**
  - Taxi parameters (`openingFee`, `costPerKm`) are read from `dataset/bedirhan.json`.
  - Taxi is used:
    - From user location to nearest public transport stop (if distance > 3 km).
    - Optional direct taxi route from start to target.
  - Visualizes the direct taxi route on the map.

- **Passenger types & discounts**
  - **Student (`Ogrenci`)**: 50% discount  
  - **Elderly (`Yasli`)**: 30% discount  
  - **General (`Genel`)**: no discount  
  - Passenger type affects the base public transport fare.

- **Payment methods**
  - **Cash (`Nakit`)**: no discount, no commission  
  - **KentKart**: 20% discount  
  - **Credit card (`KrediKarti`)**: +1.5% commission  
  - Final payable amount is computed based on both passenger type and payment method.

- **Terminal-like log panel**
  - Route details and pricing steps are logged into a `RichTextBox` at the top of the main form.

---

## Tech Stack

- **Language**: C#
- **Framework**: .NET (Windows Forms)
- **UI**: WinForms (`System.Windows.Forms`)
- **Map Library**: [GMap.NET](https://greatmaps.codeplex.com/)
- **JSON Handling**: `System.Text.Json`
- **Core Concepts**:
  - Object-oriented modeling of **vehicles, passengers, payment methods**
  - **Graph** data structure for the transport network
  - **Dijkstra-like shortest path algorithm**

---

## Project Structure

- `main.cs`  
  - Entry point (`Main` method, `[STAThread]`).
  - Reads `dataset/bedirhan.json` using `jsonreader` / `json`.
  - Creates:
    - Bus stops (`Otobus`) and tram stops (`Tramvay`)
    - Two graphs:
      - `Graph graph` (weighted by fare)
      - `Graph graph2` (weighted by time)
  - Adds nodes and edges for:
    - Buses (`Otobus`) based on `NextStops`
    - Trams (`Tramvay`) based on `NextStops`
  - Starts the main form (`Form1`) and, after user enters coordinates, calls `yakinDurakBul.EnYakinDuragiBul(...)`.

- `Graph.cs`  
  - `Graph` class with:
    - `AdjacencyList<Arac, List<(Arac, double)>>`
    - `AddNode`, `AddEdge` (bidirectional)
    - `GetShortestPath(start, end)` (Dijkstra-like)
    - `GetEdgeWeight(from, to)` for fare/time summation.

- `NextStop.cs`  
  - `NextStop` data model (StopId, distance, time, fare) – conceptual representation of a connection between stops.

- `Models/Arac/Arac.cs`  
  - Abstract base class for all vehicles:
    - `id`, `name`, `type`, `lat`, `lon`, `sonDurak`.

- `Models/Arac/Otobus.cs`  
  - `Otobus : Arac`
  - Has `List<string> NextStops` (raw next stop info like `stopId(distance, time, fare)`).

- `Models/Arac/Tramvay.cs`  
  - `Tramvay : Arac`
  - Has `List<string> NextStops` (same concept as for `Otobus`).

- `Models/Arac/Taksi.cs`  
  - `Taksi : Durak, Mesafe, Ucret`
  - Reads taxi configuration from `dataset/bedirhan.json` (`openingFee`, `costPerKm`).
  - Calculates distance between coordinates and fee for the taxi ride.

- `Models/Durak.cs`  
  - Interfaces:
    - `Durak` (stop information)
    - `Mesafe` (distance calculation)
    - `Ucret` (fare calculation).

- `Models/Yolcu/Yolcu.cs`  
  - Abstract `Yolcu` with `indirimOrani` and virtual `UcretHesapla(double ucret)`.

- `Models/Yolcu/Ogrenci.cs`, `Yasli.cs`, `Genel.cs`  
  - Implement different discount ratios for passenger types.

- `Models/Odeme/Odeme.cs`  
  - Abstract base class for payment methods with `indirimOrani`, `komisyonOrani` and `Hesapla(double toplamMaliyet)`.

- `Models/Odeme/KentKart.cs`, `KrediKarti.cs`, `Nakit.cs`  
  - Implement payment-specific discount or commission logic.

- `Form1.cs` (in `HaritaUygulamasi` namespace inside `main.cs`)  
  - Main visual form:
    - `GMapControl` for map
    - Textboxes for coordinates input
    - Buttons:
      - `Rota Hesapla` (Calculate route)
      - `Ödeme Yöntemi Seç` (Select payment method)
      - `Yolcu Türü Seç` (Select passenger type)
    - `DrawRoutes(...)`:
      - Draws public transport routes (Graph/Graph2) on the map.
      - Draws the direct taxi route.
      - Computes and logs the final fare based on passenger type & payment method.

---

## Data Source

The project expects a JSON file at:

dataset/bedirhan.jsonExample structure (simplified):

{
  "duraklar": [
    {
      "lat": 40.0,
      "lon": 29.0,
      "id": "STOP1",
      "sonDurak": false,
      "type": "Otobus",
      "name": "Otogar",
      "nextStops": [
        {
          "stopId": "STOP2",
          "mesafe": 2.5,
          "sure": 10,
          "ucret": 5.0
        }
      ],
      "transfer": {
        "transferStopId": "TRAM1",
        "transferSure": 5,
        "transferUcret": 2.0
      }
    }
  ],
  "taxi": {
    "openingFee": 10.0,
    "costPerKm": 15.0
  }
}> **Note:** The actual JSON file used in the project (`bedirhan.json`) must match the properties accessed in `main.cs` and `Taksi.cs`.

---

## Getting Started

### Prerequisites

- **Windows OS**
- **.NET (Windows Forms support)** – e.g.:
  - .NET Framework with WinForms, or
  - .NET 6/7 Windows Desktop (depending on how the project is configured in `UlasimRotaPlanlama.csproj`)
- Visual Studio or any IDE that supports WinForms development.
- **GMap.NET** references must be correctly added to the project:
  - `GMap.NET.Core`
  - `GMap.NET.WindowsForms`

### Setup

1. **Clone / Copy the project**

  
   git clone <this-repo-url>
   cd UlasimRotaPlanlama
   2. **Place the dataset**

   - Ensure `dataset/bedirhan.json` exists and follows the expected structure.

3. **Open the solution**

   - Open `UlasimRotaPlanlama.sln` in Visual Studio.

4. **Restore NuGet packages**

   - Right-click on the solution → `Restore NuGet Packages`, make sure GMap.NET is installed.

5. **Build the project**

   - Build the solution in `Debug` or `Release` mode.

6. **Run**

   - Set `UlasimRotaPlanlama` as startup project (if not already).
   - Run the application (F5).

---

## Usage

1. **Start the application.**

2. **Inspect the map**
   - Bus and tram stops are drawn on the map automatically on form load.

3. **Enter coordinates**
   - Fill in:
     - `Mevcut Enlem (lat)`
     - `Mevcut Boylam (lon)`
     - `Hedef Enlem (lat)`
     - `Hedef Boylam (lon)`
   - Click **"Rota Hesapla"**.
   - These coordinates are used to find the nearest start and target stops.

4. **Select passenger type**
   - Click **"Yolcu Türü Seç"**.
   - Choose:
     - 1 – Student
     - 2 – Elderly
     - 3 – General

5. **Select payment method**
   - Click **"Ödeme Yöntemi Seç"**.
   - Choose:
     - 1 – Cash
     - 2 – KentKart (20% discount)
     - 3 – Credit Card (+1.5% commission)

6. **View results**
   - Two public transport routes and one taxi route are drawn.
   - The terminal area displays:
     - Route details (stop names, coordinates)
     - Total base fare
     - Discounted / adjusted fare based on passenger and payment type
     - Direct taxi route fare

---

## Possible Improvements

- Add validation and error handling for missing/invalid JSON data.
- Allow dynamic loading and editing of stops and routes from the UI.
- Introduce multi-criteria optimization (time vs. cost vs. transfers).
- Support localization (English/UI translations).
- Persist user preferences (default passenger type, default payment method).

---

## License

This project does not currently specify a license.  
If you plan to publish or share it, consider adding a suitable open-source license (MIT, Apache-2.0, etc.).
