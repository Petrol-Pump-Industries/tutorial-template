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
