/api/update_settings/
=====================

.. seealso:: :any:`/limits`, :any:`data_key`.

.. note:: The list of allowed fields and their accepted args:

  * privacy.messages

    * True, False, 1, 0

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <access_token>"}

  JSON
    .. code-block:: json

      {
        "<any allowed field>": "<any allowed data>"
      }

Response (200)
  .. code-block:: json

    {
      "msg": "OK"
    }

Example request
###############

.. note:: Needed packages:

  - requests
  - cryptography

.. code-block:: python

  import requests
  import base64
  from cryptography.hazmat.primitives.asymmetric import padding
  from cryptography.hazmat.primitives import hashes
  from cryptography.hazmat.primitives.serialization import (
    load_pem_private_key,
    load_pem_public_key
  )

  username = "your_username"  # Minimum 2, maximum 32 chars.
  password = b"Your_secure_pa$$w0rd"

  to_update_settings_fields = {'privacy.messages': False}

  # Loading our private key.
  # Check /api/register for info how to generate private key.
  my_private_key = load_pem_private_key(
    open("my_private_key.pem", "rb").read(), password
  )

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

  json = {"username": username, "password": base64.b64encode(encrypted_password)}

  r2 = requests.post(host + "/login/", json=json)
  if r2.status_code != 200:
    print('\t--- POST /login ---\nHeaders:', r2.headers)
    if r2.content: print("\nJson:", r2.json(), '\n')
    raise Exception("/login request returned non-200 status code.")

  data_json = r2.json()
  access_token = data_json['access_token']
  refresh_token = data_json['refresh_token']

  print('--- Tokens ---\nAccess token: %s\nRefresh token: %s\n' % (access_token,
  refresh_token))

  headers = {"Authorization": "Bearer " + access_token}
  json = {k: v for k,v in to_update_settings_fields.items()}

  r3 = requests.post(host + "/update_settings/", json=json, headers=headers)
  if r3.status_code != 200:
    print('\t--- POST /login ---\nHeaders:', r3.headers)
    if r3.content: print("\nJson:", r3.json(), '\n')
    raise Exception("/update_settings request returned non-200 status code.")

  print("\t--- Success ---")
  print(r3.json())
