Features
========

Flask-Security allows you to quickly add common security mechanisms to your
Flask application. They include:


Session Based Authentication
----------------------------

Session based authentication is fulfilled entirely by the `Flask-Login`_
extension. Flask-Security handles the configuration of Flask-Login automatically
based on a few of its own configuration values and uses Flask-Login's
`alternative token`_ feature for remembering users when their session has
expired. Flask-Security uses ``fs_uniquifier`` from its Token Authentication
Feature (see below) to implement Flask-Login's `alternative token`_. `Flask-WTF`_
integrates with the session as well to provide out of the box CSRF support.
Flask-Security extends that to support requiring CSRF for requests that are
authenticated via session cookies, but not for requests authenticated using tokens.


Role/Identity Based Access
--------------------------

Flask-Security implements very basic role management out of the box. This means
that you can associate a high level role or multiple roles to any user. For
instance, you may assign roles such as `Admin`, `Editor`, `SuperUser`, or a
combination of said roles to a user. Access control is based on the role name and/or
permissions contained within the role;
and all roles should be uniquely named. This feature is implemented using the
`Flask-Principal`_ extension. As with basic RBAC, permissions can be assigned to roles
to provide more granular access control. Permissions can be associated with one or
more roles (the RoleModel contains a list of permissions). The values of
permissions are completely up to the developer - Flask-Security simply treats them
as strings.
If you'd like to implement even more granular access
control (such as per-object), you can refer to the Flask-Principal `documentation on this topic`_.


Password Hashing
----------------

Password hashing is enabled with `passlib`_. Passwords are hashed with the
`bcrypt`_ function by default but you can easily configure the hashing
algorithm. You should **always use a hashing algorithm** in your production
environment. Hash algorithms not listed in ``SECURITY_PASSWORD_SINGLE_HASH``
will be double hashed - first an HMAC will be computed, then the selected hash
function will be used. In this case - you must provide a ``SECURITY_PASSWORD_SALT``.
A good way to generate this is::

    secrets.SystemRandom().getrandbits(128)

Bear in mind passlib does not assume which
algorithm you will choose and may require additional libraries to be installed.

Password Validation and Complexity
-----------------------------------
Consult :ref:`pass_validation_topic`.


Basic HTTP Authentication
-------------------------

Basic HTTP authentication is achievable using a simple view method decorator.
This feature expects the incoming authentication information to identify a user
in the system. This means that the username must be equal to their email address.


Token Authentication
--------------------

Token based authentication is enabled by retrieving the user auth token by
performing an HTTP POST with a query param of ``include_auth_token`` with the authentication details
as JSON data against the
authentication endpoint. A successful call to this endpoint will return the
user's ID and their authentication token. This token can be used in subsequent
requests to protected resources. The auth token is supplied in the request
through an HTTP header or query string parameter. By default the HTTP header
name is `Authentication-Token` and the default query string parameter name is
`auth_token`. Authentication tokens are generated using a uniquifier field in the
user's UserModel. If that field is changed (via :meth:`.UserDatastore.set_uniquifier`)
then any existing authentication tokens will no longer be valid. Changing
the user's password will not affect tokens.

Note that prior to release 3.3.0 or if the UserModel doesn't contain the ``fs_uniquifier``
attribute the authentication tokens are generated using the user's password.
Thus if the user changes his or her password their existing authentication token
will become invalid. A new token will need to be retrieved using the user's new
password. Verifying tokens created in this way is very slow.

Two-factor Authentication (alpha)
----------------------------------------
Two-factor authentication is enabled by generating time-based one time passwords
(Tokens). The tokens are generated using the users `totp secret`_, which is unique
per user, and is generated both on first login, and when changing the two-factor
method (doing this causes the previous totp secret to become invalid). The token
is provided by one of 3 methods - email, sms (service is not provided), or
an authenticator app such as Google Authenticator, LastPass Authenticator, or Authy.
By default, tokens provided by the authenticator app are
valid for 2 minutes, tokens sent by mail for up to 5 minute and tokens sent by
sms for up to 2 minutes. The QR code used to supply the authenticator app with
the secret is generated using the PyQRCode library.
This feature is marked alpha meaning that backwards incompatible changes
might occur during minor releases. While the feature is operational, it has these
known limitations:

    * Limited and incomplete JSON support
    * Not enough documentation to use w/o looking at code

.. _unified-sign-in:

Unified Sign In
---------------
**This feature is in Beta - mostly due to it being brand new and little to no production soak time**

Unified sign in provides a generalized login endpoint that takes an `identity`
and a `passcode`; where (based on configuration):

    * `identity` is any of :py:data:`SECURITY_USER_IDENTITY_ATTRIBUTES` (e.g. email, username, phone)
    * `passcode` is a password or a one-time code (delivered via email, SMS, or authenticator app)

Please see this `Wikipedia`_ article about multi-factor authentication.

Using this feature, it is possible to not require the user to have a stored password
at all, and just require the use of a one-time code. The mechanisms for generating
and delivering the one-time code are similar to common two-factor mechanisms.

This one-time code can be configured to be delivered via email, SMS or authenticator app -
however be aware that NIST does not recommend email for this purpose (though many web sites do so)
due to the fact that a) email may travel through
many different servers as part of being delivered - and b) is available from any device.

Using SMS or an authenticator app means you are providing "something you have" (the mobile device)
and either "something you know" (passcode to unlock your device)
or "something you are" (biometric passcode to unlock your device).
This effectively means that using a one-time code to sign in, is in fact already two-factor (if using
SMS or authenticator app). Many large authentication providers already offer this - here is
`Microsoft's`_ version.

Note that by configuring :py:data:`SECURITY_US_ENABLED_METHODS` an application can
use this endpoint JUST with identity/password or in fact disallow passwords altogether.

`Current Limited Functionality`:

    * The Unified signin endpoint does not currently support 2FA. While this isn't really
      important for SMS and authenticator authentication methods, it would be useful for
      password and email confirmation methods.
    * Change password does not work if a user registers without a password. However
      forgot-password will allow the user to set a new password.
    * Registration and Confirmation only work with email - so while you can enable multiple
      authentication methods, you still have to register with email.

Email Confirmation
------------------

If desired you can require that new users confirm their email address.
Flask-Security will send an email message to any new users with a confirmation
link. Upon navigating to the confirmation link, the user will be automatically
logged in. There is also view for resending a confirmation link to a given email
if the user happens to try to use an expired token or has lost the previous
email. Confirmation links can be configured to expire after a specified amount
of time.


Password Reset/Recovery
-----------------------

Password reset and recovery is available for when a user forgets his or her
password. Flask-Security sends an email to the user with a link to a view which
they can reset their password. Once the password is reset they are automatically
logged in and can use the new password from then on. Password reset links  can
be configured to expire after a specified amount of time.


User Registration
-----------------

Flask-Security comes packaged with a basic user registration view. This view is
very simple and new users need only supply an email address and their password.
This view can be overridden if your registration process requires more fields.


Login Tracking
--------------

Flask-Security can, if configured, keep track of basic login events and
statistics. They include:

* Last login date
* Current login date
* Last login IP address
* Current login IP address
* Total login count


JSON/Ajax Support
-----------------

Flask-Security supports JSON/Ajax requests where appropriate. Please
look at :ref:`csrftopic` for details on how to work with JSON and
Single Page Applications. More specifically
JSON is supported for the following operations:

* Login requests
* Unigied signin requests
* Registration requests
* Change password requests
* Confirmation requests
* Forgot password requests
* Passwordless login requests
* Two-factor login requests
* Change two-factor method requests

In addition, Single-Page-Applications (like those built with Vue, Angular, and
React) are supported via customizable redirect links.

Command Line Interface
----------------------

Basic `Click`_ commands for managing users and roles are automatically
registered. They can be completely disabled or their names can be changed.
Run ``flask --help`` and look for users and roles.


.. _Click: https://palletsprojects.com/p/click/
.. _Flask-Login: https://flask-login.readthedocs.org/en/latest/
.. _Flask-WTF: https://flask-wtf.readthedocs.io/en/stable/csrf.html
.. _alternative token: https://flask-login.readthedocs.io/en/latest/#alternative-tokens
.. _Flask-Principal: https://pypi.org/project/Flask-Principal/
.. _documentation on this topic: http://packages.python.org/Flask-Principal/#granular-resource-protection
.. _passlib: https://passlib.readthedocs.io/en/stable/
.. _totp secret: https://passlib.readthedocs.io/en/stable/narr/totp-tutorial.html#overview
.. _bcrypt: https://en.wikipedia.org/wiki/Bcrypt
.. _PyQRCode: https://pypi.python.org/pypi/PyQRCode/
.. _Wikipedia: https://en.wikipedia.org/wiki/Multi-factor_authentication
.. _Microsoft's: https://docs.microsoft.com/en-us/azure/active-directory/user-help/user-help-auth-app-overview
