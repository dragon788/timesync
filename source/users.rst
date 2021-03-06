.. _users:

===============
User Management
===============

Users are managed primarily through the API, by admin users.

.. contents::

-----------

Users Model
-----------

For reference of the user fields, see the :ref:`model<model>` docs.

Here is an example user object:

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-15",
    "deleted_at": null,
    "active": true,
    "meta": "extra metadata about user"
  }

.. note::

    Usernames may consist of uppercase and lowercase letters, decimal digits, hyphen,
    period, underscore, and tilde characters.

.. note::

    Usernames are considered case-insensitive: they will be displayed using the
    case they were created with, but both the ``/users/:username`` endpoints
    and the ``/login`` endpoint will match on any capitalization, i.e. if I
    have a user named 'User1', ``GET /users/User1``, ``GET /users/user1``, and
    ``GET /users/uSeR1`` will all return that user.

-----------

Admin Users
-----------

A site-wide admin user is defined by the boolean ``admin`` field. Admins
are able to access any endpoint, and manage all users (including being the only
users which can promote others to admin status).

---------------

/users Endpoint
---------------

User management involves a new endpoint, at ``/users``. In most respects it is
similar to the other endpoints, with certain key differences.

GET /users
~~~~~~~~~~

Returns a list of all user objects.

.. code-block:: javascript

  [
    {
      "display_name": "User One",
      "username": "user1",
      "email": "user1@example.org",
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": null,
      "active": true,
      "meta": "extra metadata about user"
    },
    {
      // ...
    },
    // ...
  ]

.. note::

  The /users endpoint also includes the ability to ?include_deleted objects.

.. note::

  Usernames are permanent, even upon deletion.

GET /users/:username
~~~~~~~~~~~~~~~~~~~~

Returns a single user object.

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-15",
    "deleted_at": null,
    "active": true,
    "meta": "extra metadata about user"
  }

GET /users?include_deleted=true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  [
    {
      "display_name": "User One",
      "username": user1,
      "email": "user1@example.org",
      "site_spectator": true,
      "site_manager": false,
      "site_admin": false,
      "created_at": "2016-02-15",
      "updated_at": "2016-02-15",
      "deleted_at": "2017-06-21",
      "active": false,
      "meta": "extra metadata about user"
    },
    {
      // ...
    },
    // ...
  ]

GET /users/:username?include_deleted=true
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-15",
    "deleted_at": "2017-06-21",
    "active": false,
    "meta": "extra metadata about user"
  }

POST /users
~~~~~~~~~~~

Create a new user.

Request:

.. code-block:: javascript

  {
    "displayname": "X. Ample User",
    "username": "example",
    "password": "password",
    "email": "example@example.com"
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "active": true,
    "meta": "Some metadata about the user"
  }

Response:

.. code-block:: javascript

  {
    "displayname": "X. Ample User",
    "username": "example",
    "email": "example@example.com"
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "active": true,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-15",
    "deleted_at": null,
    "active": true,
    "meta": "Some metadata about the user"
  }

.. caution::

  It is the client's responsibility to hash the password before sending it to
  this endpoint, unlike the /login endpoint (see :ref:`auth<auth>`). The
  password should be hashed with bcrypt, using 10 rounds. The bcrypt prefix
  should be "2a" (different implementations may use different prefixes, but the
  API requires consistency for authentication).

.. note::

  This endpoint may only be accessed by admins and sitewide managers.

.. note::

  It is recommended that admins provide the user with a temporary password
  and have the user change the password when they log in.

~~~~~~~~~~~~~~~~~~~~~

POST /users/:username
~~~~~~~~~~~~~~~~~~~~~

Original object:

.. code-block:: javascript

  {
    "display_name": "User One",
    "username": "user1",
    "email": "user1@example.org",
    "site_spectator": true,
    "site_manager": false,
    "site_admin": false,
    "active": true,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-15",
    "deleted_at": null,
    "active": false,
    "meta": "extra metadata about user"
  }

Request body (made by a ``site_admin`` user):

.. code-block:: javascript

  {
    "display_name": "New Displayname",
    "password": "Battery Staple",
    "email": "user1+new@example.org",
    "meta": "Different metadata about user1",
    "site_spectator": true,
    "site_manager": true,
    "site_admin": false,
  }

The response will be:

.. code-block:: javascript

  {
    "display_name": "New Displayname",
    "username": "user1",
    "email": "user1+new@example.org",
    "site_spectator": true,
    "site_manager": true,
    "site_admin": false,
    "created_at": "2016-02-15",
    "updated_at": "2016-02-18",
    "deleted_at": null,
    "meta": "Different metadata about user1"
  }

.. caution::

  It is the client's responsibility to hash the password before sending it to
  this endpoint, unlike the /login endpoint (see :ref:`auth<auth>`). The
  password should be hashed with bcrypt, using 10 rounds. The bcrypt prefix
  should be "2a" (different implementations may use different prefixes, but the
  API requires consistency for authentication).

.. note::

  Site-wide admins can modify other users' site_spectator, site_manager, and site_admin
  fields.

  Site-wide managers can modify other users' site_spectator fields.

This endpoint may be accessed by admins, sitewide managers, or the user who is being
updated. However, users may not set their own permissions unless they are an admin, and
managers may *only* set the ``site_spectator`` field; thus the ``site_admin`` and
``site_manager`` fields may only be set by an admin.

DELETE /users/:username
~~~~~~~~~~~~~~~~~~~~~~~

Soft-delete a user. Returns a 200 OK with empty response body on success, or an
:ref:`error<errors>` on failure. Only accessible to admins.

For more information on deletion, see the DELETE section of the :ref:`API<api>` docs.

---------------

Role Management
---------------

Role management is handled through the ``projects`` and ``users`` endpoints.

The user object contains the ``site_spectator``, ``site_manager``, and
``site_admin`` fields, which are booleans designating those permissions. As
stated above, a sitewide manager may promote a user to sitewide spectator or
demote sitewide spectators; a sitewide admin may also promote a user to
sitewide manager or to admin, or demote sitewide managers or other admins (including
themselves).

The project object contains a ``users`` object, which map users (by username)
to their permissions on the project. An admin, sitewide manager, or project
manager may set these at any time, adding to or removing from any of the lists.
A project may have zero or more of members, spectators, and managers; if a
project has no managers, sitewide managers and admins may still manage the
project.
