/api/send_message/
==================

.. seealso:: :any:`/limits`, :any:`data_key`.

Payload
  Headers
    .. code-block:: json

      {"Authorization": "Bearer <access_token>"}

  JSON
    .. code-block:: json

      {
        "username": "<another user's username>",
        "message": "<encrypted message>",
        "signature": "<message signature>",
        "key": "<message's secret key>"
      }

Response (200)
  .. code-block:: json

    {
      "msg": "OK",
    }

Example request
###############

.. note:: Needed packages:

  - requests
  - secrets
  - cryptography
.. hidden-code-block:: python
  :starthidden: False

  import requests
  import base64
  import secrets
  from cryptography.fernet import Fernet
  from cryptography.hazmat.primitives.asymmetric import padding
  from cryptography.hazmat.primitives import hashes
  from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
  from cryptography.hazmat.primitives.serialization import (
      load_pem_private_key,
      load_pem_public_key
  )

  username = "your_username"  # Minimum 2, maximum 32 chars.
  password = b"Your_secure_pa$$w0rd"

  to_send_username = "another_user"
  to_send_message = "Hello!"

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

  json = {"username": to_send_username}

  r3 = requests.post(host + "/start_conversation/", json=json, headers=headers)
  if r3.status_code != 200:
    print('\t--- POST /start_conversation ---\nHeaders:', r3.headers)
    if r3.content: print("\nJson:", r3.json(), '\n')
    raise Exception("/start_conversation request returned non-200 status code.")

  another_user_public_key = load_pem_public_key(
    r3.json()['msg_public_key'].encode(), None
  )

  # Keep these random!
  msg_password = secrets.token_bytes(192)
  salt = secrets.token_bytes(64)
  kdf = PBKDF2HMAC(
    algorithm=hashes.SHA512(),
    length=32,
    salt=salt,
    iterations=100_001,
  )

  key = base64.urlsafe_b64encode(kdf.derive(msg_password))
  f = Fernet(key)
  token = f.encrypt(to_send_message.encode())

  signature = my_private_key.sign(
    to_send_message.encode(),
    padding.PSS(
      mgf=padding.MGF1(hashes.SHA512()),
      salt_length=padding.PSS.MAX_LENGTH,
    ),
    hashes.SHA512(),
  )

  encrypted_msg_password = another_user_public_key.encrypt(
    msg_password + salt,
    padding.OAEP(
      mgf=padding.MGF1(algorithm=hashes.SHA512()),
      algorithm=hashes.SHA512(),
      label=None,
    )
  )

  json = {
    "username": to_send_username,
    "message": token.decode('ascii'),
    "signature": base64.b64encode(signature),
    "key": base64.b64encode(encrypted_msg_password)
  }

  r4 = requests.post(host + "/send_message", json=json, headers=headers)
  if r4.status_code != 200:
    print('\t--- POST /send_message ---\nHeaders:', r4.headers)
    if r4.content: print("\nJson:", r4.json(), '\n')
    raise Exception("/send_message request returned non-200 status code.")
  print('\t--- Success ---')
  if r4.content: print(r4.json())


Example script response
#######################

.. code-block:: python

  '''
    --- Data-key ---

  -----BEGIN PUBLIC KEY-----
  MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAplf75PkdTbm5p/RoUOGh
  UVTCMw/6Ajs8h/FLgi1wUALCryZbu/iwWdzlT83r3IjTll0XxG94Ouh9PJi/k4sZ
  /ZmDPksG1GxGZGgzE2edVjrIHbdxGuowxD3s4mD8pjGAKd6Dypa8Y8M8G0IDaiEO
  2YNjFEFzZD/titlFPwwgZu9OP9BeRaeM6BbhrA7pgSUAmIJL61OOngzAo3jyg4ss
  eVqO7MJp2Yw1homB8OZbcbVlUX/htS11qeH3yJVI29ZL2/yuUT5ywCLdcFtcRxQh
  +qMBLJoidED1TNizH9DN7ouVyKLT4F2o25/vmLiC5cgXYuDIozcWxKU95lOIXVIz
  /tPQSeLu/tXSLPJz8DXMU4JICkLNicFrePdBPZZW8j1eCnGCp3FpNXsEH9l/pQFF
  fWegYE+E2OjSPWCR6SA953UB54SCbGKEQEq2RPkZptWw0HH/AOhSutt7HY68oQyD
  E491QLv5nEvtN/wpFw+VoY0g/VI4mW+HpeU7hjREFNLQCyOupLuzL4PeX6q2+jLn
  Ml5wehZBiaUNSxP7jTq3ie8rvWgUAEsSW0RSJ8FD8aQU7yNsMJim9DBf3edDa0L3
  L4M7IjxTI0AKPr6aWWsJ7Wgsfbzp9EILdN7CsAQB9zeV7jfiVENQHqSdCCU9xTrP
  LcgYTs40Qwqcy8ZTqh9ksS0CAwEAAQ==
  -----END PUBLIC KEY-----


  --- Tokens ---
  Access token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTYxNDg0NTk4M
  ywianRpIjoiMjFhMmM1ZDYtMjU2Yy00N2Y4LTg2NjEtNTE0Mzk5Y2VmODkzIiwibmJmIjoxNjE0ODQ
  1OTgzLCJ0eXBlIjoiYWNjZXNzIiwic3ViIjoieW91cl91c2VybmFtZSIsImV4cCI6MTYxNDg0Nzc4M
  30.EEntsPXpnQ29cW3ofUnnsGxg_cAmF4RW46rgzv0n_-E
  Refresh token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTYxNDg0NTk4M
  ywianRpIjoiYzdjZWNmOGEtOThkMi00MDFlLWIwNmEtNWEyZDNiYTU4YWZhIiwibmJmIjoxNjE0ODQ
  1OTgzLCJ0eXBlIjoicmVmcmVzaCIsInN1YiI6InlvdXJfdXNlcm5hbWUiLCJleHAiOjE2MTc0Mzc5O
  DN9.I_6eZMgX6OK5eMLzB5rLxjDJ68Qj9l0sLL3JdcGqkrE

    --- Success ---
  {'msg': 'OK'}
  '''
