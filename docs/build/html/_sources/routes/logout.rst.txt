/api/logout/
=================

.. seealso:: :any:`/limits`.

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <access_token>"}

Response (200)
  .. code-block:: json

    {
      "msg": "Access token has been revoked.",
    }

Example request
###############

.. note:: Needed packages:

  - requests
.. code-block:: python

  import requests


  host = "http://localhost:5000/api"  # Api host, must contain http/https.
  access_token = ""

  headers = {"Authorization": "Bearer %s" % access_token}
  r = requests.post(host + "/logout/", headers=headers)

  print('\t--- POST /logout ---\nStatus code:', r.status_code)
  if r.content: print("\nJson:", r.json())
