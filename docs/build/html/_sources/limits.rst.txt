Limits and conditions
=====================


POST
----

* :any:`routes/data_key` ``3 per 60 minutes``
* :any:`routes/register` ``5 per 1 minute, 10 per 30 minutes``

.. note::
  * the amount of characters of the username must be 2 < x <= 32
  * the username can't contain spaces and dots

  * the password must contain at least:
      * 10 characters
      * upper-case and lower-case letters
      * one digit
      * one special character ``!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~``
  * the amount of characters of the password must be <= 128
  * the password can't contain non-ascii characters

* :any:`routes/login` ``5 per minute, 10 per 30 minutes``

.. note::
  the password must be encrypted.

* :any:`routes/logout` ``5 per minute``
* :any:`routes/refresh_token` ``5 per minute``
* :any:`routes/start_conversation` ``10 per 15 minutes``
* :any:`routes/send_message` ``5 per 1 second``
* :any:`routes/update_settings` ``10 per 1 minute``
* :any:`routes/ack` ``5 per second``

GET
----

* :any:`routes/get_messages` ``10 per 1 minute``

Websockets
----------

* :any:`routes/notifications` ``5 listeners per user``
