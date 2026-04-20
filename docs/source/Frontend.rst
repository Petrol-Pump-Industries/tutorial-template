Frontend
========

Overview
--------
This front end uses various scripts such as pages and model. 
In order to keep each and every part of the program be easy to Read.

Fonts
-----

//temp

Colours
-----
//temp

pages
------

Login_page.dart
---------------

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
--------------

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
-------------

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