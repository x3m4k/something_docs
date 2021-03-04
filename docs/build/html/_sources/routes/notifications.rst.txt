/api/notifications/
===================

Receive notifications ASAP.

.. seealso:: :any:`/limits`, :any:`data_key`.

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <access_token>"}

Response (when new event occurs on server-side)
  .. code-block:: json

    {
      "type": "<any event type>",
      "data": "<event object data>",
      "id": "<notification id>"
    }

.. note::

  Types
    * message

  Data objects
    message
      .. code-block:: json

        {"data": {"username": "<sender's username>"}, "id": "<notification id>"}

Example request+websocket
#########################

.. note:: Needed packages:

  - requests
  - cryptography
  - websockets

.. code-block:: python

  import requests
  import base64
  import websockets
  import asyncio
  from cryptography.hazmat.primitives.asymmetric import padding
  from cryptography.hazmat.primitives import hashes
  from cryptography.hazmat.primitives.serialization import (
    load_pem_private_key,
    load_pem_public_key
  )

  username = "your_username"  # Minimum 2, maximum 32 chars.
  password = b"Your_secure_pa$$w0rd"

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

  async def connect():
    uri = "ws://%s/notifications/" % host.split('//', 1)[1]

    async with websockets.connect(
      uri, extra_headers=[("Authorization", "Bearer %s" % access_token)]
    ) as websocket:
      res = await websocket.recv()
      print('\t--- New data ---\n')
      print(res)

    await connect()  # Reconnecting after response

  asyncio.run(connect())


Example script response
#######################

.. code-block:: python

  '''
  ...

    --- New data ---

  [{"type": "message", "data": {"username": "another_user"}, "id": "1"}]
  '''
