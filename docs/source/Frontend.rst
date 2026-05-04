Frontend
========

Overview
--------
This front end uses various scripts such as pages and models. 
In order to keep each and every part of the program be easy to Read.

Fonts
-----

//temp

Colours
-----
//temp

Main.dart
---------

.. code-block:: dart

  import 'package:flutter/material.dart';
  import 'package:here_sdk/core.dart';
  import 'package:here_sdk/core.engine.dart';
  import 'src/pages/main_page.dart';
  import 'src/pages/login_page.dart';
  import 'src/pages/register_page.dart';
  import 'src/repositories/auth_service.dart';
  import 'src/widgets/Navigation_menu.dart';
  import 'src/utils/db_init_desktop.dart'
      if (dart.library.html) 'src/utils/db_init_web.dart';


The program starts from this script. It initializes the HERE SDK, checks for an existing user session, and launches the app with the appropriate initial route (either the map page if a session exists or the login page if not).

Pages
------

Login_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  Future<void> _onLogin() async {
    if (!_formKey.currentState!.validate()) return;
    setState(() => _isLoading = true);
    try {
      await _authService.login(
        _emailController.text.trim(),
        _passwordController.text,
      );
      if (mounted) Navigator.of(context).pushReplacementNamed('/map');
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.toString().replaceFirst('Exception: ', '')),
            backgroundColor: Colors.redAccent,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

This function is used to handle the login process.
It validates the form, shows a loading indicator,
and attempts to log in using the provided email and password.
If successful, it navigates to the map page; if there's an error, it displays a snackbar with the error message.

.. code-block:: dart

    Future<void> _onGuest() async {
        setState(() => _isLoading = true);
        try {
        await _authService.createGuestAccount();
        if (mounted) Navigator.of(context).pushReplacementNamed('/map');
        } finally {
        if (mounted) setState(() => _isLoading = false);
        }
    }

This function allows users to create a guest account and navigate to the map page without needing to log in with an email and password. It also shows a loading indicator while the account is being created.

main_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  void _onMenuItemSelected(NavigationMenuItem item) {
    final nextRoute = _routeByItem[item];
    if (nextRoute == null) {
      return;
    }

    if (ModalRoute.of(context)?.settings.name == nextRoute) {
      return;
    }

    Navigator.of(context).pushReplacementNamed(nextRoute);
  }

This function handles the selection of menu items in the navigation drawer. It checks if the selected item has a corresponding route and if it's not already the current route, it navigates to the new route using `pushReplacementNamed`.

map_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  class MapPage extends StatelessWidget {
    const MapPage({super.key, this.hereInitMessage, this.journey});

    final String? hereInitMessage;
    final Journey? journey;

    @override
    Widget build(BuildContext context) {
      return MapUi(initializationMessage: hereInitMessage, journey: journey);
    }
  }

This class represents the Map Page of the application. It takes an optional initialization message and a journey object, which are passed to the Map UI component for rendering the map and related information.

register_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  Future<void> _onRegister() async {
    if (!_formKey.currentState!.validate()) return;
    setState(() => _isLoading = true);
    try {
      await _authService.register(
        _emailController.text.trim(),
        _usernameController.text.trim(),
        _passwordController.text,
      );
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('Account created! Please log in.'),
            backgroundColor: Colors.green,
          ),
        );
        Navigator.of(context).pop(); // Back to login
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.toString().replaceFirst('Exception: ', '')),
            backgroundColor: Colors.redAccent,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

This function handles the user registration process. It validates the form, shows a loading indicator, and attempts to register a new account using the provided email, username, and password. If registration is successful, it displays a success message and navigates back to the login page. If there's an error during registration, it shows an error message in a snackbar.

search_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart



settings_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  Future<void> _onChangePassword() async {
    final newPasswordController = TextEditingController();
    final confirmPasswordController = TextEditingController();
    
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Change Password'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: newPasswordController,
              decoration: const InputDecoration(labelText: 'New Password'),
              obscureText: true,
            ),
            TextField(
              controller: confirmPasswordController,
              decoration: const InputDecoration(labelText: 'Confirm New Password'),
              obscureText: true,
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () {
              if (newPasswordController.text == confirmPasswordController.text && 
                  newPasswordController.text.isNotEmpty) {
                Navigator.pop(context, true);
              } else {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Passwords do not match!')),
                );
              }
            },
            child: const Text('Change'),
          ),
        ],
      ),
    );

    if (confirmed == true && _currentUser != null) {
      setState(() => _isUpdating = true);
      try {
        await _authService.updatePassword(_currentUser!.id!, newPasswordController.text);
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Password changed successfully!')),
        );
      } catch (e) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Failed to change password: $e')),
        );
      } finally {
        if (mounted) setState(() => _isUpdating = false);
      }
    }
  }


the function `_onChangePassword` is responsible for handling the password change process. It prompts the user to enter a new password and confirm it. If the passwords match and are not empty, it proceeds with the password change; otherwise, it shows an error message.

.. code-block:: dart


  Future<void> _onLogout() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Logout'),
        content: const Text('Are you sure you want to log out?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          TextButton(
            style: TextButton.styleFrom(foregroundColor: Colors.red),
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Logout'),
          ),
        ],
      ),
    );

    if (confirmed == true) {
      await _authService.logout();
      if (mounted) Navigator.of(context).pushReplacementNamed('/login');
    }
  }

The function `_onLogout` handles the user logout process. It first shows a confirmation dialog to ensure the user wants to log out. If the user confirms, it calls the logout method from the authentication service and navigates back to the login page.

trips_page.dart
^^^^^^^^^^^^^^^

.. code-block:: dart

  Future<void> _loadJourneys() async {
    setState(() => _isLoading = true);
    try {
      final user = await _authService.getCurrentUser();
      if (user != null && user.id != null) {
        final journeys = await _journeyRepo.readAllJourneys(user.id!);
        setState(() => _journeys = journeys);
      } else {
        setState(() => _journeys = []);
      }
    } catch (e) {
      debugPrint('Error loading journeys: $e');
      setState(() => _journeys = []);
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

The function `_loadJourneys` is responsible for loading the user's journeys from the repository. It first sets a loading state, then retrieves the current user and their journeys. If successful, it updates the state with the retrieved journeys; if there's an error, it logs the error and sets the journeys to an empty list. Finally, it resets the loading state.

vehicles_page.dart
^^^^^^^^^^^^^^^^^^

.. code-block:: dart

   class VehiclesPage extends StatelessWidget {
     const VehiclesPage({super.key});

     @override
     Widget build(BuildContext context) {
       return const VehiclesMenu();
     }
   }

This class represents the Vehicles Page of the application. It simply returns a `VehiclesMenu` widget, which is responsible for displaying the available vehicles and related options to the user.

models
------

classes that represent sets of data that we concistantly use throughout the program

account_model.dart
^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  class Account {
  final int? id;
  final String email;
  final String username;
  final String passwordHash; // never store plain text
  final bool isGuest;
  final String? profilePicture; // Base64 or local file path
  final bool isLoggedIn;
  final DateTime? createdAt;
  final DateTime? updatedAt;

  Account({
    this.id,
    required this.email,
    required this.username,
    required this.passwordHash,
    this.isGuest = false,
    this.profilePicture,
    this.isLoggedIn = false,
    this.createdAt,
    this.updatedAt,
  });

  Map<String, Object?> toMap() {
    return {
      'id': id,
      'email': email,
      'username': username,
      'password_hash': passwordHash,
      'is_guest': isGuest ? 1 : 0,
      'is_logged_in': isLoggedIn ? 1 : 0,
      'profile_picture': profilePicture,
      'created_at': createdAt?.toIso8601String(),
      'updated_at': updatedAt?.toIso8601String(),
    };
  }

  static Account fromMap(Map<String, dynamic> map) {
    return Account(
      id: map['id'] as int?,
      email: map['email'] as String,
      username: map['username'] as String,
      passwordHash: map['password_hash'] as String,
      isGuest: (map['is_guest'] as int?) == 1,
      isLoggedIn: (map['is_logged_in'] as int?) == 1,
      profilePicture: map['profile_picture'] as String?,
      createdAt: map['created_at'] != null
          ? DateTime.parse(map['created_at'] as String)
          : null,
      updatedAt: map['updated_at'] != null
          ? DateTime.parse(map['updated_at'] as String)
          : null,
    );
  }

  Account copy({
    int? id,
    String? email,
    String? username,
    String? passwordHash,
    bool? isGuest,
    String? profilePicture,
    bool? isLoggedIn,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) =>
      Account(
        id: id ?? this.id,
        email: email ?? this.email,
        username: username ?? this.username,
        passwordHash: passwordHash ?? this.passwordHash,
        isGuest: isGuest ?? this.isGuest,
        profilePicture: profilePicture ?? this.profilePicture,
        isLoggedIn: isLoggedIn ?? this.isLoggedIn,
        createdAt: createdAt ?? this.createdAt,
        updatedAt: updatedAt ?? this.updatedAt,
      );
  }

This class represents the Account model, which contains information about a user's account.

favourites_location_model.dart
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  class FavoriteLocation {
    final int? id;
    final int? userId;
    final String name;
    final String? address;
    final double latitude;
    final double longitude;
    final String? placeId;
    final String? iconType;
    final DateTime? createdAt;

    //...

    GeoCoordinates toGeoCoordinates() {
      return GeoCoordinates(latitude, longitude);
    }
  }

this is the FavoriteLocation model, which represents a user's saved lcoation with various details.

journey_model.dart
^^^^^^^^^^^^^^^^

.. code-block:: dart
  class Journey {
  final int? id;
  final int? userId;
  final int? vehicleId;
  final Location from;
  final Location to;
  final List<Location>? waypoints;
  final double? distanceMiles;
  final double? durationMinutes;
  final double? cost;
  final double? fuelLitres;
  final DateTime? createdAt;
  final DateTime? updatedAt;

  //...

  Map<String, Object?> toMap() {
    final fromMap = from.toMap();
    final toMap = to.toMap();
    final nowIso = DateTime.now().toIso8601String();
    final createdIso = createdAt?.toIso8601String() ?? nowIso;
    final updatedIso = updatedAt?.toIso8601String() ?? createdIso;
    return {
      'id': id,
      'user_id': userId,
      'vehicle_id': vehicleId,
      // journey_history schema stores JSON text for locations.
      'origin': jsonEncode(fromMap),
      'destination': jsonEncode(toMap),
      'waypoints': waypoints != null ? jsonEncode(waypoints!.map((w) => w.toMap()).toList()) : null,
      'distance_miles': distanceMiles,
      'duration_minutes': durationMinutes,
      'cost': cost,
      'fuel_litres': fuelLitres,
      // 'date' is the legacy column name from early schema versions (NOT NULL).
      // We always populate it so old installed databases don't throw a constraint error.
      'date': createdIso,
      'created_at': createdIso,
      'updated_at': updatedIso,
    };
  }

This class represents the Journey model which contains details about the user's journey allowing them to view how much ca journey may cost.


location_model.dart
^^^^^^^^^^^^^^^^

.. code-block:: dart

  class Location {
  final int? id;
  final String name;
  final double latitude;
  final double longitude;
  final String? address;
  final String? placeId;

  //...

  /// Create Location from map (database/API response)
  static Location fromMap(Map<String, dynamic> map) {
    return Location(
      id: map['id'] as int?,
      name: map['name'] as String? ?? 'Unknown',
      latitude: map['latitude'] as double? ?? 0.0,
      longitude: map['longitude'] as double? ?? 0.0,
      address: map['address'] as String?,
      placeId: map['place_id'] as String?,
    );
  }


  /// Convert to HERE SDK GeoCoordinates for map integration
  /// This allows us to easily use Location instances with HERE SDK features
  /// Here SDK uses a different coordinate system, so this conversion is necessary for map display and routing
  GeoCoordinates toGeoCoordinates() {
    return GeoCoordinates(latitude, longitude);
  }

This model represents a Location with various details such as name, coordinates, and address. It also includes a method to convert the location to HERE SDK's GeoCoordinates for map integration.

vehicle_model.dart
^^^^^^^^^^^^^^^^

.. code-block:: dart

  class Vehicle {
  final int? id;
  final int? userId;
  final String? registrationNumber;
  final String make;
  final String? model;
  final String colour;
  final int? engineCapacity;
  final String taxStatus;
  final String motStatus;
  final int? year;
  final String? gearType;
  final int? mpg;
  final int? co2Emissions;
  final int? engineSize;
  final String fuelType;
  final String? nickname;
  final DateTime? createdAt;
  final DateTime? updatedAt;

  //...

  factory Vehicle.fromJson(Map<String, dynamic> json) {
    return Vehicle(
      registrationNumber: json['registrationNumber'] ?? '',
      make: json['make'] ?? 'Unknown',
      model: json['model'],
      year: json['year'],
      gearType: json['gearType'],
      mpg: json['mpg'],
      co2Emissions: json['co2Emissions'],
      engineSize: json['engineSize'],
      colour: json['colour'] ?? 'Unknown',
      engineCapacity: json['engineCapacity'],
      fuelType: json['fuelType'] ?? 'Unknown',
      taxStatus: json['taxStatus'] ?? 'Unknown',
      motStatus: json['motStatus'] ?? 'Unknown',
      nickname: json['nickname'],
      isActive: json['isActive'],
      createdAt: json['createdAt'] != null ? DateTime.tryParse(json['createdAt'] as String) : null,
      updatedAt: json['updatedAt'] != null ? DateTime.tryParse(json['updatedAt'] as String) : null,
    );
  }

This model represents a Vehicle with various details such as make, model, year, and fuel type. It includes a factory constructor to create a Vehicle instance from a JSON map, which is useful for parsing API responses or database records.

Widgets
-------

We found that we had to re-use widgets a few time or rather separated widgets from thier associated pages so we can layer the code better. As a result we can fix widgets and code separatly and easily.

add_vehicle_flow.dart
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  // ...

      super.initState();
    if (widget.initialVehicle != null) {
      _fetchedVehicle = widget.initialVehicle;
      _isManualMode = true; // Allow editing existing data
      _currentStep = 1;
      _regController.text = _fetchedVehicle!.registrationNumber ?? '';
      _nicknameController.text = _fetchedVehicle!.nickname ?? '';
      _mpgController.text = _fetchedVehicle!.mpg?.toString() ?? '';
      
      // Pre-fill manual controllers too
      _makeController.text = _fetchedVehicle!.make;
      _modelController.text = _fetchedVehicle!.model ?? '';
      _colourController.text = _fetchedVehicle!.colour;
      _fuelController.text = _fetchedVehicle!.fuelType;
      _yearController.text = _fetchedVehicle!.year?.toString() ?? '';
    }
  }

  //...

    Future<void> _lookupVehicle() async {
    if (_regController.text.isEmpty) return;

    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      final vehicle = await _dvlaRepo.getVehicleDetails(_regController.text);
      if (mounted) {
        setState(() {
          _fetchedVehicle = vehicle;
          _currentStep = 1;
          _isLoading = false;
          
          // Pre-fill manual controllers in case they want to edit
          _makeController.text = _fetchedVehicle!.make;
          _modelController.text = _fetchedVehicle!.model ?? '';
          _colourController.text = _fetchedVehicle!.colour;
          _fuelController.text = _fetchedVehicle!.fuelType;
          _yearController.text = _fetchedVehicle!.year?.toString() ?? '';
        });
      }
    } on DvlaNotFoundException catch (_) {
      setState(() {
        _error = "Vehicle not found. Please check your Registration Number.\n(Tip: Are you mistaking '0' for 'O' or '1' for 'I'?)";
        _isLoading = false;
      });
    } on DvlaRateLimitException catch (_) {
      setState(() {
        _error = "Too many requests. Please wait a moment and try again.";
        _isLoading = false;
      });
    } on DvlaException catch (e) {
      setState(() {
        _error = e.message;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = "An unexpected error occurred: $e";
        _isLoading = false;
      });
    }
  }

  Future<void> _saveVehicle() async {
    if (_fetchedVehicle == null) return;
    
    final nickname = _nicknameController.text;
    if (nickname.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("Please enter a nickname for your vehicle")),
      );
      return;
    }

    if (nickname.length > 10) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("Nickname must be 10 characters or less")),
      );
      return;
    }

    final user = await _authService.getCurrentUser();
    if (user == null || user.id == null) return;

    final isUnique = await _localRepo.isNicknameUnique(nickname, user.id!);
    // Only check uniqueness if the nickname changed OR if it's a new vehicle
    if (!isUnique && (widget.initialVehicle == null || widget.initialVehicle!.nickname != nickname)) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("You already have a vehicle with this nickname")),
      );
      return;
    }

    setState(() => _isLoading = true);

    try {
      final vehicleToSave = _fetchedVehicle!.copy(
        userId: user.id,
        nickname: nickname,
        registrationNumber: _regController.text,
        make: _isManualMode ? _makeController.text : _fetchedVehicle!.make,
        model: _isManualMode ? _modelController.text : _fetchedVehicle!.model,
        colour: _isManualMode ? _colourController.text : _fetchedVehicle!.colour,
        fuelType: _isManualMode ? _fuelController.text : _fetchedVehicle!.fuelType,
        year: _isManualMode ? int.tryParse(_yearController.text) : _fetchedVehicle!.year,
        mpg: int.tryParse(_mpgController.text),
        isActive: widget.initialVehicle?.isActive ?? false,
      );

      if (widget.initialVehicle != null) {
        await _localRepo.update(vehicleToSave);
      } else {
        await _localRepo.create(vehicleToSave);
      }
      widget.onVehicleAdded();
      if (mounted) Navigator.pop(context);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Failed to save vehicle: $e")),
      );
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }


  Widget _buildRegInput() {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        const Text(
          "Enter your vehicle's Registration Number to automatically fetch details.",
          textAlign: TextAlign.center,
        ),
        const SizedBox(height: 20),
        TextField(
          controller: _regController,
          decoration: InputDecoration(
            labelText: "Registration Number",
            hintText: "e.g. AB12 CDE",
            border: const OutlineInputBorder(),
            errorText: _error,
            suffixIcon: _isLoading 
              ? const Padding(
                  padding: EdgeInsets.all(12),
                  child: CircularProgressIndicator(strokeWidth: 2),
                )
              : IconButton(
                  icon: const Icon(Icons.search),
                  onPressed: _lookupVehicle,
                ),
          ),
          textCapitalization: TextCapitalization.characters,
          onSubmitted: (_) => _lookupVehicle(),
        ),
        const SizedBox(height: 16),
        const Text("OR", style: TextStyle(fontWeight: FontWeight.bold, fontSize: 12, color: Colors.grey)),
        const SizedBox(height: 8),
        OutlinedButton.icon(
          onPressed: _switchToManualMode,
          icon: const Icon(Icons.edit_note),
          label: const Text("Enter Details Manually"),
        ),
        if (_error != null) ...[
          const SizedBox(height: 20),
          const Text(
            "Is the DVLA service down? You can also try adding it manually.",
            style: TextStyle(fontSize: 12, color: Colors.grey),
          ),
        ],
      ],
    );
  }

This widget represents the flow for adding a vehicle, allowing users to either look up their vehicle details using the DVLA API or enter them manually. It includes error handling for various scenarios such as not finding the vehicle or hitting rate limits, and ensures that the nickname for the vehicle is unique and valid before saving.

map_ui.dart
^^^^^^^^^^

.. code-block:: dart

  Future<Position?> _getCurrentPosition() async {
    final hasAccess = await _canAccessDeviceLocation();
    if (!hasAccess) {
      return null;
    }

    return Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
    );
  }

  Future<GeoCoordinates?> _getCurrentCoordinates() async {
    final position = await _getCurrentPosition();
    if (position == null) {
      return null;
    }

    // Convert the Geolocator Position to HERE SDK GeoCoordinates
    return GeoCoordinates(position.latitude, position.longitude);
  }

  //...

  Future<void> _moveCameraToInitialPosition(
    HereMapController controller,
  ) async {
    GeoCoordinates center;
    try {
      center =
          await _getCurrentCoordinates() ??
          GeoCoordinates(widget.initialLatitude, widget.initialLongitude);
    } catch (_) {
      center = GeoCoordinates(widget.initialLatitude, widget.initialLongitude);
    }

    controller.camera.lookAtPoint(center);
  }

  void _followCameraToPosition(Position position) {
    final controller = _mapController;
    if (controller == null) {
      return;
    }

    final target = GeoCoordinates(position.latitude, position.longitude);
    
    if (_isDrivingMode) {
      // Perspective mode for driving: Tilt and Rotate to Match Heading
      controller.camera.lookAtPointWithGeoOrientationAndMeasure(
        target,
        GeoOrientationUpdate(position.heading, 60), // 60 degree tilt
        MapMeasure(MapMeasureKind.zoomLevel, 17),  // Close zoom for driving
      );
    } else {
      // Standard top-down follow
      controller.camera.lookAtPoint(target);
    }
  }

This code snippet includes functions for obtaining the user's current location and moving the map camera to that location. It uses the Geolocator package to get the device's position and converts it to HERE SDK's GeoCoordinates. The camera can be set to follow the user's position in either a standard top-down view or a tilted perspective mode for driving.

Navigation_Menu.dart
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  Widget build(BuildContext context) {
    return SafeArea(
      top: false,
      child: SizedBox(
        height: 94,
        child: Stack(
          clipBehavior: Clip.none,
          alignment: Alignment.topCenter,
          children: [
            Positioned.fill(
              top: 16,
              child: Container(
                padding: const EdgeInsets.only(bottom: 6),
                decoration: BoxDecoration(
                  color: Theme.of(context).colorScheme.surface,
                  border: Border(
                    top: BorderSide(
                      color: Theme.of(
                        context,
                      ).dividerColor.withValues(alpha: 0.2),
                    ),
                  ),
                ),
                child: Row(
                  children: [
                    Expanded(
                      child: _NavItem(
                        icon: Icons.receipt_long_outlined,
                        selectedIcon: Icons.receipt_long,
                        label: 'Trips',
                        selected: currentItem == NavigationMenuItem.tripHistory,
                        onTap: () => onItemSelected(NavigationMenuItem.tripHistory),
                      ),
                    ),
                    Expanded(
                      child: _NavItem(
                        icon: Icons.map_outlined,
                        selectedIcon: Icons.map,
                        label: 'Map',
                        selected: currentItem == NavigationMenuItem.map,
                        onTap: () =>
                            onItemSelected(NavigationMenuItem.map),
                      ),
                    ),
                    const SizedBox(width: 72),
                    Expanded(
                      child: _NavItem(
                        icon: Icons.directions_car_outlined,
                        selectedIcon: Icons.directions_car,
                        label: 'Vehicles',
                        selected: currentItem == NavigationMenuItem.vehicles,
                        onTap: () => onItemSelected(NavigationMenuItem.vehicles),
                      ),
                    ),
                    Expanded(
                      child: _NavItem(
                        icon: Icons.settings_outlined,
                        selectedIcon: Icons.settings,
                        label: 'Settings',
                        selected: currentItem == NavigationMenuItem.settings,
                        onTap: () => onItemSelected(NavigationMenuItem.settings),
                      ),
                    ),
                  ],
                ),
              ),
            ),
            Positioned(
              top: 0,
              child: Material(
                color: Colors.transparent,
                child: InkWell(
                  customBorder: const CircleBorder(),
                  onTap: () {
                    onSearchPressed?.call();
                    onItemSelected(NavigationMenuItem.search);
                  },
                  child: Ink(
                    width: 58,
                    height: 58,
                    decoration: const ShapeDecoration(
                      shape: CircleBorder(),
                      color: Colors.orange,
                    ),
                    child: const Icon(
                      Icons.search,
                      color: Colors.white,
                      size: 30,
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  }

this widget represents the custom navigation menu used in the application. It includes navigation items for Trips, Map, Vehicles, and Settings, as well as a central search button. The menu is designed to be visually distinct and user-friendly, with clear icons and labels for each navigation option.

saved_location_popup.dart
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    Widget build(BuildContext context) {
    return Dialog(
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      child: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text(
                    'Saved Locations',
                    style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                  ),
                  IconButton(
                    icon: const Icon(Icons.close),
                    onPressed: () => Navigator.of(context).pop(),
                  ),
                ],
              ),
              const Divider(),
              if (_isLoading)
                const Padding(
                  padding: EdgeInsets.all(24.0),
                  child: CircularProgressIndicator(),
                )
              else if (_favorites.isEmpty)
                Padding(
                  padding: const EdgeInsets.symmetric(vertical: 32.0),
                  child: Column(
                    children: [
                      const Icon(
                        Icons.favorite_border,
                        size: 48,
                        color: Colors.grey,
                      ),
                      const SizedBox(height: 16),
                      const Text(
                        'No saved locations yet.',
                        style: TextStyle(color: Colors.grey),
                      ),
                      const SizedBox(height: 16),
                      FilledButton.icon(
                        onPressed: () {
                          Navigator.of(context).pop();
                          widget.onAddNew();
                        },
                        icon: const Icon(Icons.add),
                        label: const Text('Add a location'),
                      ),
                    ],
                  ),
                )
              else
                Flexible(
                  child: ListView.builder(
                    shrinkWrap: true,
                    itemCount: _favorites.length,
                    itemBuilder: (context, index) {
                      final fav = _favorites[index];
                      return ListTile(
                        leading: const Icon(Icons.location_on),
                        title: Text(fav.name),
                        subtitle: fav.address != null
                            ? Text(
                                fav.address!,
                                maxLines: 1,
                                overflow: TextOverflow.ellipsis,
                              )
                            : null,
                        trailing: IconButton(
                          icon: const Icon(
                            Icons.delete_outline,
                            color: Colors.red,
                          ),
                          onPressed: () => _deleteFavorite(fav),
                        ),
                        onTap: () {
                          Navigator.of(context).pop(fav.toLocation());
                        },
                      );
                    },
                  ),
                ),
              if (_favorites.isNotEmpty) ...[
                const Divider(),
                TextButton.icon(
                  onPressed: () {
                    Navigator.of(context).pop();
                    widget.onAddNew();
                  },
                  icon: const Icon(Icons.add),
                  label: const Text('Add new location'),
                ),
              ],
            ],
          ),
        ),
      ),
    );
  }
}

This widget represents a popup dialog that displays the user's saved locations. It allows users to view their saved locations, delete them, or add new ones. If there are no saved locations, it shows a friendly message and prompts the user to add a new location.

vehicle_menu.dart
^^^^^^^^^^^^^^^^

.. code-block:: dart


  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    return Scaffold(
      appBar: AppBar(title: const Text('My Vehicles')),
      body: _vehicles.isEmpty
          ? _buildEmptyState()
          : ListView(
              padding: const EdgeInsets.all(16),
              children: [
                _buildTipBanner(),
                const SizedBox(height: 16),
                ..._vehicles.map((vehicle) => _buildVehicleCard(vehicle)),
              ],
            ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _showAddVehicleFlow,
        icon: const Icon(Icons.add),
        label: const Text("Add Vehicle"),
      ),
    );
  }

  Widget _buildTipBanner() {
    final theme = Theme.of(context);
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: theme.colorScheme.primaryContainer.withOpacity(0.3),
        borderRadius: BorderRadius.circular(16),
        border: Border.all(color: theme.colorScheme.primary.withOpacity(0.1)),
      ),
      child: Row(
        children: [
          Container(
            padding: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: theme.colorScheme.primary.withOpacity(0.1),
              shape: BoxShape.circle,
            ),
            child: Icon(
              Icons.tips_and_updates_outlined,
              color: theme.colorScheme.primary,
              size: 20,
            ),
          ),
          const SizedBox(width: 16),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  "Quick Tip",
                  style: theme.textTheme.labelMedium?.copyWith(
                    color: theme.colorScheme.primary,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  "Use the Active toggle on a vehicle card to choose your default vehicle.",
                  style: theme.textTheme.bodySmall?.copyWith(
                    color: theme.colorScheme.onSurfaceVariant,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildEmptyState() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            Icons.directions_car_outlined,
            size: 64,
            color: Colors.grey[400],
          ),
          const SizedBox(height: 16),
          const Text(
            "No vehicles added yet",
            style: TextStyle(fontSize: 18, color: Colors.grey),
          ),
          const SizedBox(height: 8),
          const Text(
            "Add your car to start calculating trip costs!",
            style: TextStyle(color: Colors.grey),
          ),
          const SizedBox(height: 24),
          ElevatedButton(
            onPressed: _showAddVehicleFlow,
            child: const Text("Add My First Vehicle"),
          ),
        ],
      ),
    );
  }

  Widget _buildVehicleCard(Vehicle vehicle) {
    final theme = Theme.of(context);
    final isActive = _selectedVehicle?.id == vehicle.id;

    return Card(
      elevation: isActive ? 4 : 1,
      margin: const EdgeInsets.only(bottom: 12),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
        side: isActive
            ? BorderSide(color: theme.colorScheme.primary, width: 2)
            : BorderSide.none,
      ),
      child: InkWell(
        onTap: () => _onSelectVehicle(vehicle),
        borderRadius: BorderRadius.circular(12),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          vehicle.nickname ?? "Unnamed Vehicle",
                          style: theme.textTheme.titleLarge?.copyWith(
                            fontWeight: FontWeight.bold,
                            color: theme.colorScheme.primary,
                          ),
                        ),
                        Text(
                          vehicle.registrationNumber?.toUpperCase() ??
                              "UNKNOWN REG",
                          style: theme.textTheme.bodyMedium?.copyWith(
                            fontWeight: FontWeight.w500,
                            letterSpacing: 1.2,
                          ),
                        ),
                      ],
                    ),
                  ),
                  if (isActive)
                    Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 8,
                        vertical: 4,
                      ),
                      decoration: BoxDecoration(
                        color: theme.colorScheme.primaryContainer,
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: Text(
                        "ACTIVE",
                        style: TextStyle(
                          fontSize: 10,
                          fontWeight: FontWeight.bold,
                          color: theme.colorScheme.onPrimaryContainer,
                        ),
                      ),
                    ),
                  IconButton(
                    icon: const Icon(Icons.edit_outlined, color: Colors.grey),
                    onPressed: () => _onEditVehicle(vehicle),
                  ),
                  IconButton(
                    icon: const Icon(Icons.delete_outline, color: Colors.red),
                    onPressed: () => _onDeleteVehicle(vehicle),
                  ),
                ],
              ),
              const Divider(height: 24),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  _buildStat(
                    Icons.directions_car,
                    "${vehicle.make} ${vehicle.model ?? ''}",
                  ),
                  _buildStat(Icons.local_gas_station, vehicle.fuelType),
                  _buildStat(Icons.bolt, "${vehicle.mpg ?? '--'} MPG"),
                ],
              ),
              const SizedBox(height: 12),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'Use as active vehicle',
                    style: theme.textTheme.bodyMedium?.copyWith(
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                  Switch.adaptive(
                    value: isActive,
                    onChanged: (value) {
                      if (value) {
                        _onSelectVehicle(vehicle);
                      }
                    },
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildStat(IconData icon, String label) {
    return Row(
      children: [
        Icon(icon, size: 16, color: Colors.grey),
        const SizedBox(width: 4),
        Text(
          label,
          style: const TextStyle(fontSize: 12, fontWeight: FontWeight.w500),
        ),
      ],
    );
  }
  }


this widget represents the menu for managing vehicles in the application. It allows users to view their added vehicles, set an active vehicle, and add new vehicles. The UI includes an empty state when no vehicles are added and a tip banner to guide users on how to use the active vehicle feature. Each vehicle is displayed in a card with options to edit or delete it.

services
--------

navigation_service.dart
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  import '../models/journey_model.dart';

  /// A singleton service that persists the active navigation journey across
  /// page changes, so that switching tabs doesn't kill the active route.
  class NavigationService {
    NavigationService._internal();
    static final NavigationService instance = NavigationService._internal();

    // The journey that is currently being navigated. Null means no active session.
    Journey? activeJourney;

    // Whether navigation/driving mode is currently active.
    bool isNavigationActive = false;

    /// Start a new navigation session with the given journey.
    void startNavigation(Journey journey) {
      activeJourney = journey;
      isNavigationActive = true;
    }

    /// End the current navigation session and clear all state.
    void stopNavigation() {
      activeJourney = null;
      isNavigationActive = false;
    }
  }

The comments explain each line in detail but the overall purpose of this script is to manage active change in your loaction.

Utilites
--------

db_init_desktop.dart
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

  import 'dart:io';
  import 'package:sqflite_common_ffi/sqflite_ffi.dart';

  void initDatabaseForPlatform() {
    if (Platform.isWindows || Platform.isLinux) {
      sqfliteFfiInit();
      databaseFactory = databaseFactoryFfi;
    }
  }


