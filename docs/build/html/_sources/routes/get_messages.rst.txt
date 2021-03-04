/api/get_messages/
==================

.. seealso:: :any:`/limits`.

Payload
  Headers
  .. code-block:: json

    {"Authorization": "Bearer <access_token>"}

  JSON
    .. code-block:: json

      {
        "username": "another_user",
        "from_ts": 0,
        "to_ts": 0
      }

    Parameters
      - **from_ts**
          UTC timestamp from which we receive messages (>)
      - **to_ts**
          UTC timestamp within which we receive messages (<=)

Response (200)
  .. code-block::

    {
      "messages": [
        {
          "msg": "<encrypted fernet token>",
          "sign": "<message signature>",
          "key": "<encrypted message key>",
          "date": {"$date": "<UTC date format: YYYY-mm-ddThh:mm:ssZ>"}
        }, ...
      ]
    }

Example request
###############

.. note:: Needed packages:

  - requests
  - cryptography

.. hidden-code-block:: python
  :starthidden: False

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

  to_get_messages_username = "another_user"
  to_get_messages_from_ts = 0
  to_get_messages_to_ts = 0

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
  json = {
    "username": to_get_messages_username,
    "from_ts": to_get_messages_from_ts,
    "to_ts": to_get_messages_to_ts
  }

  r3 = requests.get(host + "/get_messages/", json=json, headers=headers)
  if r3.status_code != 200:
    print('\t--- POST /login ---\nHeaders:', r3.headers)
    if r3.content: print("\nJson:", r3.json(), '\n')
    raise Exception("/login request returned non-200 status code.")

  print('\t--- Got messages ---')
  print(r3.json())


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
  Access token:
  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTYxNDg0NzUyM
  ywianRpIjoiNzA1YmUwOTEtNGRhYi00ZTY2LTk1MzQtOWQxMDZhMDAxOGI3IiwibmJmIjoxNjE0ODQ
  3NTIzLCJ0eXBlIjoiYWNjZXNzIiwic3ViIjoieW91cl91c2VybmFtZSIsImV4cCI6MTYxNDg0OTMyM
  30.NN98hEwYE24xsXJ3FtIQmD2o7GR6uflR6b-JTf1bZoY
  Refresh token:
  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTYxNDg0NzUyM
  ywianRpIjoiMWUxMmMwZTUtOWIzOS00ZDAzLWJmODgtMDhlNjE3NzEzOGIzIiwibmJmIjoxNjE0ODQ
  3NTIzLCJ0eXBlIjoicmVmcmVzaCIsInN1YiI6InlvdXJfdXNlcm5hbWUiLCJleHAiOjE2MTc0Mzk1M
  jN9.Y-Clv8VJarG5tt5yGQhw_GsdgiZPi7XDp1nnhF_fUYI

    --- Got messages ---
  {'messages': [{'date': {'$date': '2021-03-04T08:44:48.862Z'}, 'key':
  'aVqznihbWFHB25aFb9wNxgGUmwzMYpqwh+hdssQHppdL2nbopP7rBzPasS0ImnjwWBTvI4xvdBKni
  BC1whyy7EJP/Ws613i0oAQkXnmcJDXAc1usWB8J0cUA9ic0H07cVoKiZdplO5u3QDBgKzIKhYaUiD7
  707MiGsXYW9X+466xdaXJoMwL89S4rji4XhJdNE15pEbxUOU8BchbOa/xpBD07uv0FKyvHvFgAaWi9
  Aiqh1oPu3DIxwfDK/g4Cv3DN3S+RqfagZXA528xU8XmGrF1vfjFuk2qDxkNz8wXZYSzwJdygoJmJ1D
  I4OQHqvDnUKeJZ4wpU70zvhZ3OdGzpn5j5371lh2i8Dx7shd1am2DViZiz6xjG7QLT2fqUu1ABtQ3x
  7PJ9fGSnIzg+5DS6HaeoW5UlTtdQbLCkUqLxi/RhmyNt2vv2R0FC80+Ww/3s/saoj96SHJpLQBqxy/
  FTlGCPUIJLZkIlCcRixVuiDYECxoMvwrsQ2k0IVlpYJvb+z2G6Y0/Ebrbb9s40KCk2vKE3Km9p93kX
  jLsH8j0oKTOxqrsAjBqt6y7fWsugahVnpincXLK/EdxiUmyy13T5c549CT2r51Gp+qpw2NRATjL0Vb
  UonoKCmneBJBvPHAmho93U25YREbnJOhNUc/VmmZtx08SU4uDS2fgPsSA8Bs=',
  'msg': 'gAAAAABgQJ4Atj8oQ0ZzgculxLBfPBpjntZeqIq91S0asfNI1lXlSVWehJNEfO5XinVO8_
  _plB9Lmal1naxXbQV5sU70P6jLWg==',
  'sign': 'JTTYK2E3BvRCT9q4JY1Eosgkc2x3QuNOCGUPUFoGgW2qC6NFxeURINgoecAqdTVMOAABC
  HJdmxg7l3mpWsxGR0TTnKcfZhDDzPAb13Bt9pC/ixHVjWMd40kgNgebVXhNX094IuwncAcx7KkODxe
  dMpjILKpygVgeQrFdLHwJu6591RgnXeXDRRUA6REiHaAzGqPBZya3LSRugrEEnK54A+CgnZPJZ9Nln
  G2b0sTxybO3VEt24cipwt7YHNoiDvs+eU9yQE8vj4MAy/k14mW2lgeQ80I11u9mUhb1B4g1Aobd+VG
  N3Fqxw6sIru69wHrLvW+t0qQ0Y457dJTW5zeddOgw6KPcPkl/1hIk1/enPll4rbNhlgM12w4JfDVH5
  7BCPAZfc1w3W/iUyeJmHJ2sWbpJ2gMaQg2ZqwsNYKwPStArQGmFSIM44PuN5v6TjmKtnaj8bHtm72Z
  dtF9f5Soqz8bPQq2dSk1h+Kz7/17N6J3ajd9d5aQrK1b4q//S8qbepb242KSCedTruRk6OuVDKGsc8
  m0JVwTo9mOcPZgeSxzalySLJ10+WuK6oY1TB3dybmnizzdHH5rtd8TZw/GpRu3Pd5lK/RuAf5WJwy7
  93J9LUfo++g4wYVRx7lseevm6mgxdBhF/a3W2dkXnwTYbBXRS0mSuUkKhkMEaDflx55g='}]}
  '''
