Backend
=======

core 
----

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

