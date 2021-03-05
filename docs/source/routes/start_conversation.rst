/api/start_conversation/
========================

.. warning:: May change soon, see `Man-in-the-middle attack <https://en.wikipedia.org/wiki/Man-in-the-middle_attack>`_.

.. seealso:: :any:`/limits`.

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <access_token>"}
  JSON
    .. code-block:: json

      {"username": "<another user's username>"}

Response (200)
  .. code-block:: json

    {
      "msg_public_key": "<user's rsa public key>"
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

  headers = {"Authorization": "Bearer %s" % access_token}
  json = {"username": "another_user"}
  r = requests.post(host + "/start_conversation/", json=json, headers=headers)

  print('\t--- POST /start_conversation ---\nStatus code:', r.status_code)
  if r.content: print("\nJson:", r.json())


Example script response
#######################

.. code-block:: python

  '''
  ...
    --- POST /start_conversation ---
  Status code: 200
  Json: {"msg_public_key": "-----BEGIN PUBLIC KEY-----
  MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAwOyba5hzzR1HjkjOLjXh
  tQYqsMChvFFlwe5SuTaknIW1g6kTsAg1fCSv8xZpSrJ8wYmBr5yNFoP++lMy1Z+k
  4H1rTtOGcihwuU/6uAM3P8FLrLMvRdUSGj6Yng8/qlcG3YJAErPmz1Za0eVi32k6
  WQyZ67jaxl8ksR0JbBkhE3Jk/wifmhw493vAdGJ5MxahOG25ZbIJPTbqMR+2nXa+
  g3M3BSTA5NM5eEwiLtft+tyg7Y0ww0D7siwxTHEOUZ67hvXlunc7LbRELaSxqPCv
  QKHyI80qfCnXmMAgZoi5Zj+69Gt21TEVnsxtQ0cQuE9S2FUyfbC30Rl3qqOLJqp1
  xUMChvkzhnhBQh/D2hTb33omvENp58XLNv0yz794x5QQJaDgx97TzHLKPCtI4zW4
  U44wdZXiphzRZ6c+ty5aoK4wtCFo+xsmMQVvU4A2WRjEA2unE2LmFfUH+5/W1clJ
  yI1YzoXXmmOInctFQn9trKFbjo4LHoZYsHhLj6sPwHDifm/l640Rk5Bu5vjkJ4yh
  kyj9ePZ+yn31n4IYt+j9VfZR9AEZttIoKoWu8x5BJDTdq7+X/PRQ4Uqtm0Y7k7oZ
  eD5wZL8pJQyUAysDmINz0Iw3e0GsPdQuO98QlVq+bMnoTRpUOgCEo7m3fm56P/DW
  DVAniuMuv8XZfOph/WwkVeUCAwEAAQ==
  -----END PUBLIC KEY-----"
  }'''
