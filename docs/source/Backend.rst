Backend
=======

core 
----

the core section contains the fundamental building blocks of the PricePump project, including configuration settings and database management. It serves as the backbone for the application's functionality, providing essential constants, paths, and database connection details that are used throughout the project.

config.py 
^^^^^^^^^

.. code-block:: python

    import os

    # --- Paths ---
    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    DATA_DIR = os.path.join(BASE_DIR, "data")
    CSV_FILE_NAME = "fuel_prices.csv"
    CSV_FILE_PATH = os.path.join(DATA_DIR, CSV_FILE_NAME)

    # --- Database ---
    DATABASE_URL = "sqlite:///./pricepump.db"

    # --- Baseline Physics Constants ---
    BASE_MPG = 40.0
    BASE_CC = 1500.0
    BASE_WEIGHT = 1400.0
    LITERS_PER_GALLON = 4.54609

    # --- Default Prices ---
    DEFAULT_FUEL_PRICE_GBP = 1.45

This script contains the configuration settings for the PricePump project, including paths, database URL, baseline physics constants, and default fuel prices. It serves as a central location for all configuration-related information, making it easier to manage and maintain the project.

database.py
^^^^^^^^^^^

.. code-block:: python

    import os

    # --- Paths ---
    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    DATA_DIR = os.path.join(BASE_DIR, "data")
    CSV_FILE_NAME = "fuel_prices.csv"
    CSV_FILE_PATH = os.path.join(DATA_DIR, CSV_FILE_NAME)

    # --- Database ---
    DATABASE_URL = "sqlite:///./pricepump.db"

    # --- Baseline Physics Constants ---
    BASE_MPG = 40.0
    BASE_CC = 1500.0
    BASE_WEIGHT = 1400.0
    LITERS_PER_GALLON = 4.54609

    # --- Default Prices ---
    DEFAULT_FUEL_PRICE_GBP = 1.45

This script contains the database configuration settings for the PricePump project, including paths, database URL, baseline physics constants, and default fuel prices. It serves as a central location for all database-related information, making it easier to manage and maintain the project.

services
--------

calculation_service.py
^^^^^^^^^^^^^^^

.. code-block:: python

    from datetime import datetime
    from typing import Optional, Tuple
    from core.config import BASE_MPG, BASE_CC, BASE_WEIGHT, LITERS_PER_GALLON

    def get_season_factor(now: datetime = None) -> Tuple[float, str]:
        """
        Returns (mpg_multiplier, season_name). Winter decreases efficiency.
        """
        if now is None:
            now = datetime.utcnow()

        month = now.month
        if month in (12, 1, 2):
            return 0.90, "Winter"
        elif month in (3, 4, 5):
            return 0.97, "Spring"
        elif month in (6, 7, 8):
            return 1.00, "Summer"
        else:
            return 0.95, "Autumn"

    ROUTE_FACTORS = {
        "motorway": 1.10,   # Cruising efficiency
        "mixed":    1.00,   # Standard default
        "city":     0.80,   # Traffic efficiency hit
    }

    def get_route_factors(route_type: str) -> float:
        return ROUTE_FACTORS.get(route_type.lower(), 1.00)

    BASE_ROUTE_SPEEDS_MPH = {
        "motorway": 62.0,
        "mixed": 42.0,
        "city": 22.0,
    }

    def estimate_journey_time_minutes(
        distance_miles: float,
        route_type: str,
        engine_cc: int,
        weight_kg: int,
        passengers: int = 0,
        cargo_kg: float = 0.0,
        roof_rack: bool = False,
        weather_factor: float = 1.0,
        vehicle_year: Optional[int] = None,
        here_base_duration_seconds: Optional[float] = None,
    ) -> Tuple[int, str]:
        """
        Estimates travel time in minutes using route baseline speed and vehicle/environment penalties.
        This keeps ETA aligned with the same factors already used by the fuel-cost model.
        """
        if distance_miles <= 0:
            return 0, "Zero-distance journey"

        route_key = route_type.lower()
        base_speed_mph = BASE_ROUTE_SPEEDS_MPH.get(route_key, BASE_ROUTE_SPEEDS_MPH["mixed"])

        # Larger engines get a mild acceleration/merging benefit; small engines can be slower under load.
        engine_bonus = max(-0.06, min(0.05, (engine_cc - BASE_CC) / 3000.0))

        # Heavier vehicle + payload tends to reduce average speed in real-world traffic.
        vehicle_mass_penalty = max(0.0, ((weight_kg - BASE_WEIGHT) / 100.0) * 0.003)
        extra_mass_kg = (passengers * 75) + cargo_kg
        payload_penalty = (extra_mass_kg / 100.0) * 0.008

        drag_penalty = 0.03 if roof_rack else 0.0

        age_penalty = 0.0
        if vehicle_year:
            age = datetime.now().year - vehicle_year
            if age > 12:
                age_penalty = min(0.07, (age - 12) * 0.004)

        # weather_factor < 1.0 means harsher conditions; translate to a speed penalty.
        weather_penalty = 0.0
        if weather_factor < 1.0:
            weather_penalty = min(0.15, (1.0 / max(weather_factor, 0.6)) - 1.0)

        total_penalty = vehicle_mass_penalty + payload_penalty + drag_penalty + age_penalty + weather_penalty
        adjusted_speed_mph = base_speed_mph * (1.0 + engine_bonus) * (1.0 - total_penalty)
        adjusted_speed_mph = max(8.0, adjusted_speed_mph)

        model_minutes = (distance_miles / adjusted_speed_mph) * 60.0
        minutes = int(round(model_minutes))
        note = (
            f"Route baseline {base_speed_mph:.0f} mph, adjusted {adjusted_speed_mph:.1f} mph "
            f"after vehicle/load/weather factors"
        )

        if here_base_duration_seconds and here_base_duration_seconds > 0:
            here_minutes = here_base_duration_seconds / 60.0
            # Keep HERE traffic signal dominant while preserving custom vehicle/weather influence.
            blended_minutes = (here_minutes * 0.7) + (model_minutes * 0.3)
            minutes = int(round(blended_minutes))
            note = (
                f"HERE base {here_minutes:.1f} min blended with custom model {model_minutes:.1f} min "
                f"(70/30 weighting)"
            )

        return minutes, note

    def calculate_fuel_consumption(
        engine_cc: int,
        weight_kg: int,
        distance_miles: float,
        dvla_mpg: Optional[float] = None,
        # Optional parameters for advanced physics
        passengers: int = 0,
        cargo_kg: float = 0.0,
        roof_rack: bool = False,
        ac_on: bool = False,
        windows_open: bool = False,
        vehicle_year: Optional[int] = None,
    ) -> Tuple[float, float, str]:
        """
        Calculates low/high fuel burner thresholds incorporating advanced degradation 
        and environmental factors.
        """
        # 1. Base Physics MPG
        cc_diff = (engine_cc - BASE_CC) / 100.0
        weight_diff = (weight_kg - BASE_WEIGHT) / 100.0
        physics_mpg = BASE_MPG - (cc_diff * 1.2) - (weight_diff * 0.6)
        
        # 2. Apply Age Degradation (approx 3% drop per 5 years after 10 years)
        current_year = datetime.now().year
        if vehicle_year:
            age = current_year - vehicle_year
            if age > 10:
                degradation = min(0.35, (age - 10) * 0.015) 
                physics_mpg *= (1.0 - degradation)

        # 3. Apply Mass/Cargo Penalty (~1.5% drop per 50kg)
        # Average passenger weight = 75kg
        total_extra_mass = (passengers * 75) + cargo_kg
        mass_penalty = (total_extra_mass / 50.0) * 0.015
        physics_mpg *= (1.0 - mass_penalty)

        # 4. Aerodynamics & A/C (The Window Crossover Point)
        # We estimate "speed" based on distance/context, but for now we look at A/C vs Windows
        if ac_on:
            physics_mpg *= 0.90 # 10% hit for A/C
        
        if roof_rack:
            physics_mpg *= 0.85 # 15% hit for drag

        if windows_open:
            # Simplistic crossover: if Motorway (implied by higher speed factors), windows are worse than A/C
            # For now, we apply a general drag penalty if windows are open
            physics_mpg *= 0.95

        physics_mpg = max(8.0, min(80.0, physics_mpg))

        if dvla_mpg and dvla_mpg > 0:
            # Cross-reference with factory specs (weighted slightly towards physics for older cars)
            liters_factory = (distance_miles / dvla_mpg) * LITERS_PER_GALLON
            liters_physics = (distance_miles / physics_mpg) * LITERS_PER_GALLON
            
            # If the car is old, prioritize physics model
            if vehicle_year and (current_year - vehicle_year) > 12:
                liters_low = liters_physics * 0.95
                liters_high = liters_physics * 1.05
                note = f"Prioritizing Physics model due to vehicle age ({physics_mpg:.1f} MPG)"
            else:
                liters_low = min(liters_factory, liters_physics)
                liters_high = max(liters_factory, liters_physics)
                note = f"Range: DVLA specs ({dvla_mpg:.0f} MPG) vs Physics model ({physics_mpg:.1f} MPG)"
        else:
            # No factory MPG; using physics model with margin
            liters_mid = (distance_miles / physics_mpg) * LITERS_PER_GALLON
            liters_low = liters_mid * 0.92
            liters_high = liters_mid * 1.08
            note = f"Physics model only ({physics_mpg:.1f} MPG estimate), includes advanced factors."

        return liters_low, liters_high, note

calculation_service.py contains the core logic for estimating fuel consumption and journey time based on various factors such as engine size, vehicle weight, route type, and environmental conditions. It incorporates advanced physics-based calculations and can cross-reference with factory specifications when available.

data_update_service.py
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    import os
    import requests
    import time
    from datetime import datetime
    from core.config import CSV_FILE_PATH

    # Placeholder for a reliable UK aggregated fuel price feed.
    # Many developers use consolidated versions of the CMA retailer feeds.
    # If you have a specific API key or URL, we can swap it here!
    DEFAULT_DATA_URL = "https://raw.githubusercontent.com/the-fuel-project/fuel-prices/main/dist/fuel-prices.csv"

    def download_latest_fuel_prices(url: str = DEFAULT_DATA_URL) -> bool:
        """
        Downloads the latest fuel price CSV and saves it to the local data folder.
        Returns True if successful.
        """
        print(f"Update: Checking for latest fuel data from {url}...")
        
        try:
            response = requests.get(url, timeout=15)
            response.raise_for_status()
            
            # Ensure the data directory exists
            os.makedirs(os.path.dirname(CSV_FILE_PATH), exist_ok=True)
            
            with open(CSV_FILE_PATH, "wb") as f:
                f.write(response.content)
                
            print(f"Update Success: New price data saved to {CSV_FILE_PATH}")
            return True
        except Exception as e:
            print(f"Update Failed: Could not fetch new prices. Error: {e}")
            return False

    def is_data_stale(max_age_hours: int = 24) -> bool:
        """
        Checks if the local fuel_prices.csv is older than the max_age_hours.
        Returns True if the file needs updating or doesn't exist.
        """
        if not os.path.exists(CSV_FILE_PATH):
            return True
        
        file_time = os.path.getmtime(CSV_FILE_PATH)
        age_seconds = time.time() - file_time
        return age_seconds > (max_age_hours * 3600)

    def auto_update_if_needed():
        """
        Triggers a download only if the current file is stale.
        """
        if is_data_stale():
            print("Update: Local fuel data is stale or missing. Updating now...")
            download_latest_fuel_prices()
        else:
            print("Update: Local fuel data is up to date (less than 24h old).")

data_update_service.py contains functions to download the latest fuel price data from a specified URL and save it locally. It also includes logic to check if the existing data is stale (older than a specified number of hours) and only triggers an update if necessary. This helps ensure that the application always has access to recent fuel price information without unnecessary downloads.

fuel_service.py
^^^^^^^^^^^^^^^^

.. code-block:: python

    def _get_live_data_for_today() -> Tuple[list, list]:
        """
        Returns (stations, prices) from in-memory/disk cache when possible.
        Refreshes from API only when today's cache is missing.
        """
        today = _today_utc_date_str()

        # Fast path: in-memory cache already for today.
        if (
            _LIVE_API_CACHE.get("refresh_date") == today
            and _LIVE_API_CACHE.get("stations")
            and _LIVE_API_CACHE.get("prices")
        ):
            return _LIVE_API_CACHE["stations"], _LIVE_API_CACHE["prices"]

        # Warm path: persisted cache for today exists.
        if _load_live_cache_from_disk_for_today(today):
            return _LIVE_API_CACHE["stations"], _LIVE_API_CACHE["prices"]

        # Cold path: fetch once for today.
        token = get_oauth_token()
        if not token:
            return [], []

        headers = {
            "Authorization": f"Bearer {token}",
            "Accept": "application/json",
        }

        try:
            stations = _fetch_all_batches("", headers)
            prices = _fetch_all_batches("/fuel-prices", headers)
            if not stations or not prices:
                return [], []

            _LIVE_API_CACHE["refresh_date"] = today
            _LIVE_API_CACHE["stations"] = stations
            _LIVE_API_CACHE["prices"] = prices
            _save_live_cache_to_disk(today, stations, prices)
            return stations, prices
        except Exception as e:
            print(f"Live API daily refresh error: {e}")
            return [], []

    // _get_live_data_for_today is a function that retrieves live fuel station and price data, utilizing an in-memory cache for fast access and a disk cache for persistence across sessions. It checks if the data for the current day is already available in memory or on disk before making an API call to fetch fresh data. This approach optimizes performance while ensuring that the application has access to up-to-date information when needed.


    def get_current_fuel_price(
        fuel_type: str,
        year: Optional[int] = None,
        user_lat: Optional[float] = None,
        user_lon: Optional[float] = None,
        radius_miles: float = 15.0,
    ) -> Tuple[float, Optional[str]]:
        """
        Uses live API as the default source, then falls back to CSV/local data.
        If GPS is provided, calculates the average of the nearest forecourts.
        """
        csv_col, suggestion = resolve_fuel_csv_column(fuel_type, year)

        # API-first: use latest data when available.
        live_price = _get_live_price_fallback(fuel_type, year, user_lat, user_lon, radius_miles)
        if live_price is not None:
            return live_price, suggestion

        if not os.path.exists(CSV_FILE_PATH):
            print(f"Warning: Price data not found at {CSV_FILE_PATH}. Using fallback default (1.45).")
            return DEFAULT_FUEL_PRICE_GBP, suggestion

        baselines = calculate_regional_averages()

        if user_lat is not None:
            sector = get_regional_sector(user_lat)
            default_price = baselines.get((sector, csv_col), DEFAULT_FUEL_PRICE_GBP)
        else:
            default_price = DEFAULT_FUEL_PRICE_GBP

        location_mode = user_lat is not None and user_lon is not None
        potential_stations = []
        national_prices = []

        try:
            with open(CSV_FILE_PATH, mode="r", encoding="utf-8") as f:
                reader = csv.DictReader(f)
                for row in reader:
                    val = row.get(csv_col, "")
                    if not val:
                        continue

                    try:
                        price = float(val)
                        if not (80 < price < 300):
                            continue

                        if location_mode:
                            row_lat_str = row.get("forecourts.location.latitude", "")
                            row_lon_str = row.get("forecourts.location.longitude", "")
                            if row_lat_str and row_lon_str:
                                row_lat = float(row_lat_str)
                                row_lon = float(row_lon_str)
                                dist = _haversine_miles(user_lat, user_lon, row_lat, row_lon)
                                if dist <= radius_miles:
                                    potential_stations.append((dist, price))
                        else:
                            national_prices.append(price)
                    except (ValueError, TypeError):
                        continue

            if location_mode and potential_stations:
                potential_stations.sort(key=lambda x: x[0])
                top_n = potential_stations[:5]
                avg_pence = sum(p for _, p in top_n) / len(top_n)
                return avg_pence / 100.0, suggestion

            if location_mode:
                sector = get_regional_sector(user_lat)
                fallback_price = baselines.get((sector, csv_col), default_price)
                return fallback_price, suggestion

            if national_prices:
                avg_pence = sum(national_prices) / len(national_prices)
                return avg_pence / 100.0, suggestion

            return default_price, suggestion
        except Exception as e:
            print(f"Error reading fuel price data: {e}")
            return default_price, suggestion

    // get_current_fuel_price is a function that retrieves the current fuel price based on the specified fuel type and optional parameters such as vehicle year and user location. It first attempts to get live data from an API, then falls back to reading from a local CSV file if the API data is unavailable. If GPS coordinates are provided, it calculates the average price of nearby fuel stations within a specified radius. If no location data is available, it uses national averages or defaults as necessary.

fuel_service.py contains the logic for retrieving current fuel prices, utilizing both live API data and local CSV data as fallbacks. It can calculate regional averages and provide location-based pricing when GPS coordinates are available. This service ensures that the application can access accurate fuel price information under various conditions.

weather_service.py
^^^^^^^^^^^^^^^^

.. code-block:: python

    import requests
        from typing import Optional, Dict

        def get_weather_data(lat: float, lon: float) -> Optional[Dict]:
            """
            Fetches current weather data for a given GPS location using Open-Meteo.
            Returns a dict with temperature, windspeed, and weathercode.
            """
            url = "https://api.open-meteo.com/v1/forecast"
            params = {
                "latitude": lat,
                "longitude": lon,
                "current_weather": "true"
            }
            
            try:
                response = requests.get(url, params=params, timeout=5)
                if response.status_code == 200:
                    return response.json().get("current_weather")
            except Exception as e:
                print(f"Weather API Error: {e}")
            
            return None

        def calculate_weather_efficiency_factor(weather_data: Optional[Dict]) -> float:
            """
            Calculates an efficiency multiplier based on weather conditions.
            1.0 is neutral. < 1.0 is less efficient.
            """
            if not weather_data:
                return 1.0
                
            factor = 1.0
            
            # 1. Temperature Impact
            temp = weather_data.get("temperature", 15) # Default to 15C
            if temp < 5:
                # Cold start / engine warming / air density
                factor *= 0.95
            elif temp > 30:
                # AC usage load
                factor *= 0.97
                
            # 2. Wind Impact
            wind_speed = weather_data.get("windspeed", 0)
            if wind_speed > 20: 
                # Significant drag increase
                factor *= 0.96
            elif wind_speed > 40:
                # Stormy conditions
                factor *= 0.90
                
            return round(factor, 3)

weather_service.py contains functions to fetch current weather data for a given GPS location using the Open-Meteo API and to calculate an efficiency multiplier based on the weather conditions. The efficiency factor can be used in fuel consumption calculations to account for the impact of temperature and wind on vehicle performance.