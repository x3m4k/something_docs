/api/refresh_token/
===================

.. seealso:: :any:`/limits`.

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <refresh_token>"}

Response (200)
  .. code-block:: json

    {
      "access_token": "<new access_token>"
    }

Example request
###############

.. note:: Needed packages:

  - requests
.. code-block:: python

  import requests


  host = "http://localhost:5000/api"  # Api host, must contain http/https.
  refresh_token = ""

  headers = {"Authorization": "Bearer %s" % refresh_token}
  r = requests.post(host + "/refresh_token/", headers=headers)

  print('\t--- POST /refresh_token ---\nStatus code:', r.status_code)
  if r.content: print("\nJson:", r.json())
