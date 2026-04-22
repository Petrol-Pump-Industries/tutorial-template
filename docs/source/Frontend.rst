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

.. code-block:: flutter
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
------------------

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
-----------------

.. code-block:: dart



settings_page.dart
------------------

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
------------------

.. code-block:: dart

vehicles_page.dart
------------------

.. code-block:: dart
