Welcome to PricePump industries!
================================

build
-----

make sure you have an up to date flutter SDK e.g visual code

begin  by cloning the githup repository (https://github.com/Petrol-Pump-Industries/PricePump.git)

once this had been done you need to download all packages in the terminal by running the command

.. code-block:: bash

   flutter pub get

in order to use the application you will also need here_sdk adn download it to a plugins folder in the root directory.

the link: https://docs.here.com/here-sdk/docs/readme

download the flutter and dart version.

youll also need to run an android emulator to load the application

once loading a modern android phone youll need to be in the right directory.

.. code-block:: bash

   cd price_pump_project

after this you'll finally be able to run the application using:

.. code-block:: bash

   flutter run

Dependancies
----------------

These are associated dependencies automatically downloaded with the pubspec.yaml file

dependencies:
  flutter:
    sdk: flutter

cupertino_icons: ^1.0.8
http: ^1.2.0
sqflite: ^2.3.0
sqflite_common_ffi: ^2.3.3
path: ^1.9.0
sqlite3: ^2.4.6
geolocator: ^12.0.0

here_sdk:
   path: plugins/heresdk-explore-flutter-4.25.5.0.274356
image_picker: ^1.2.1


.. note::

   This project is under active development.

Contents
--------

.. toctree::

   Frontend
   Backend

Lumache hosts its documentation on Read the Docs.
