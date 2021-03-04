/api/login/
=================

.. seealso:: :any:`/limits`, :any:`data_key`.

Payload
  JSON
    .. code-block:: json

      {
        "username": "<username>",
        "password": "<encrypted with data_key>"
      }

Response (200)
  .. code-block:: json

    {
      "msg": "Logged in as <username>.",
      "access_token": "<jwt_access_token>",
      "refresh_token": "<jwt_refresh_token>"
    }

Example request
###############

.. note:: Needed packages:

  - requests
  - cryptography

.. code-block:: python

  import requests
  import base64
  from cryptography.hazmat.primitives import hashes
  from cryptography.hazmat.primitives.asymmetric import padding
  from cryptography.hazmat.primitives.serialization import (
      load_pem_public_key
  )

  # ! Read docs for the information:
  # https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa.html

  username = "your_username"  # Minimum 2, maximum 32 chars.
  password = b"Your_secure_pa$$w0rd"

  host = "http://localhost:5000/api"  # Api host, must contain http/https.

  r = requests.post(host + "/data_key/", json={"username": username})
  if r.status_code != 200:
    print('\t--- POST /data_key ---\nHeaders:', r.headers)
    if r.content: print("\nJson:", r.json(), '\n')
    raise Exception("/data_key request returned non-200 status code.")

  json_data = r.json()
  key_string = json_data['key']
  print('\t--- Data-key ---\n\n%s\n' % key_string)
  data_key = load_pem_public_key(key_string.encode())

  encrypted_password = data_key.encrypt(
    password,
    padding.OAEP(
      mgf=padding.MGF1(algorithm=hashes.SHA512()),
      algorithm=hashes.SHA512(),
      label=None
    )
  )

  r2 = requests.post(
    host + "/login/",
    json={
      "username": username,
      "password": base64.b64encode(encrypted_password)
  })

  print('\t--- POST /login ---\nStatus code:', r2.status_code)
  if r2.content: print("\nJson:", r2.json())
