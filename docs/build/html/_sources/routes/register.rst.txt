/api/register/
=================

.. seealso:: :any:`/limits`, :any:`data_key`.

Payload
  JSON
    .. code-block:: json

      {
        "username": "<username>",
        "password": "<encrypted with data_key>",
        "private_key": "<user's generated private key>"
      }

Response (200)
  .. code-block:: json

    {
      "msg": "User <username> was created.",
      "access_token": "<jwt_access_token>",
      "refresh_token": "<jwt_refresh_token>"
    }

Example request
###############

.. note:: Needed packages:

  - requests
  - cryptography

1. Choose a reasonable secure password, pick a username.
2. Run python script:
  .. code-block:: python

    import base64
    import requests
    from cryptography.hazmat.backends import default_backend
    from cryptography.hazmat.primitives import hashes
    from cryptography.hazmat.primitives import serialization
    from cryptography.hazmat.primitives.asymmetric import rsa
    from cryptography.hazmat.primitives.asymmetric import padding
    from cryptography.hazmat.primitives.serialization import (
        load_pem_public_key
    )

    # ! Read docs for the information:
    # https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa.html

    def get_private_key():
      private_key = rsa.generate_private_key(
        public_exponent=65537, key_size=4096, backend=default_backend()
      )

      return private_key

    username = "your_username"  # Minimum 2, maximum 32 chars.
    password = b"Your_secure_pa$$w0rd"

    host = "http://localhost:5000/api"  # Api host, must contain http/https.

    # Generating our private key.
    our_private_key = get_private_key()

    # Then we encrypt it with our password.
    our_pem_file = our_private_key.private_bytes(
      encoding=serialization.Encoding.PEM,
      format=serialization.PrivateFormat.TraditionalOpenSSL,
      encryption_algorithm=serialization.BestAvailableEncryption(password),
    )

    # Saving the key locally for later use.
    with open("my_private_key.pem", "wb") as f:
      f.write(our_pem_file)

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
        label=None  # Must be passed
      )
    )

    # We're base64 encoding password, because we can't transfer bytes in json.
    r2 = requests.post(
      host + "/register/",
      json={
        "username": username,
        "password": base64.b64encode(encrypted_password),
        "private_key": our_pem_file.decode()
      }
    )

    print('\t--- POST /register ---\nStatus code:', r2.status_code)
    if r2.content: print("\nJson:", r2.json())

Results
#######
  - You have registered.
  - You have `access_token` and `refresh_token` tokens.
  - You have your encrypted private_key as pem file locally and in the database.

Example script response
#######################

.. code-block:: python

  '''
  --- Data-key ---

    -----BEGIN PUBLIC KEY-----
  MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA15nHOKZ48ECHJ6nnebFq
  8OHr+K+h6jVKF7UD3uKHisFqToDssUuzM+3yI887e4UStH9yeQDeVFloyT5FY5ov
  SQU3dlATi+bYGzKq3dXpVJP+9XFq0q0QJCpVAy/eBM+9OPz/xhjrhvoUPrCGqSnO
  UC+psntW2WIdLTrfzGMpaZdyWRu5u9lIjrshLe2iqm81MdaUk+lvD1Y3wRntcZ5V
  cxQkZnvPoaAO4Pfb2nOyLFywGRwb09AglWHW2XW4rKE9glSX0rbj4GWiOB5CdlOe
  wTYE9CYHYxGgdoq/jS9NGwGr5uhR2pzc7HbgYxXC0yTTI8R1fogO8HhuUaT7flzo
  2mX+igPuyzARXMXyjhEzNyzFfHtFWO+ReX2ysCMBQyEjdHGSqhu6b6tJQIMYzg55
  w1Ci3OUd1a60AeipR1GcQl8A2twJHVPMEAkwKoF1lQQ4Y4fC7oc8++qBASJctcR7
  8gUmllr14DBx+jLwzkDqoS5ezIOYvIl9X08KJlSQAB7YRT7AbSSKFShlCf8F3FLK
  Ryp1UXX18PGJLZdXmmhngupA44UC4f88F8CikziS8SIZCvqZBoQiCurTNwjS0yg5
  y7vGTzoJOGftBatF62ljHYaDnuZzblMypWOwqI2C9OdiBS6xUllFbgp0Y6MF9Yzz
  GJZauxVaVV3DBTV5UIbftFkCAwEAAQ==
  -----END PUBLIC KEY-----

    --- POST /register ---
  Status code: 200

  Json: {
    'access_token': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2Us' \
    'ImlhdCI6MTYxNDgzNTI5MCwianRpIjoiNWIyNjMwNmYtN2JmNy00NTUzLTk3YzItYTI4ZTcwOD' \
    'EzNGM0IiwibmJmIjoxNjE0ODM1MjkwLCJ0eXBlIjoiYWNjZXNzIiwic3ViIjoieW91cl91c2Vy' \
    'bmFtZSIsImV4cCI6MTYxNDgzNzA5MH0.Bc2Oz_-m8lARPIMRAiLje24tbIaVjcaxkSOhzuR8vgE',
    'msg': 'User your_username was created.',
    'refresh_token': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2Us' \
    'ImlhdCI6MTYxNDgzNTI5MCwianRpIjoiOTM5ZGQwZTgtNjU2NC00NTM3LWE1MWEtMGI1ZTg3ZW' \
    M4ZWZlIiwibmJmIjoxNjE0ODM1MjkwLCJ0eXBlIjoicmVmcmVzaCIsInN1YiI6InlvdXJfdXNlc' \
    m5hbWUiLCJleHAiOjE2MTc0MjcyOTB9.eQZBeZ6V8S7QsqvHFmgYK4TVmLPZnjXEMAcTySkNHuU'}
  '''
