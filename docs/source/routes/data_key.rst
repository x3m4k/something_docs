/api/data_key/
=================

You'll need to this key before you can send data (login/registration).

.. seealso:: :any:`/limits`, :any:`register`, :any:`login`.

.. warning:: Key is valid until specified date (the ``until`` field)!

Payload
  JSON
    .. code-block:: json

      {"username": "your_username"}

Response (200)
  .. code-block:: json

    {"key": "<public_key>", "until": "<UTC date format: YYYY-mm-ddThh:mm:ssZ>"}

Example response (200)
  .. code-block:: json

    {"key": "-----BEGIN PUBLIC KEY-----
    MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAxwibJ7vrG8GS7HF+BM7I
    hAkma+5OCBcAUDW7QtwubLMh3or3u+/XMLOLdL80fmKmP7zn3w6XvujavFRMbvyN
    BtPW2vmBy4HbyLnS8xEETsc9pzpnbOoG1IGH8Cu9RrCoI0PCGQvkBW7Sws7lGZeL
    IXAgEZcj6Cxfi6bMPgEYX9NuvHyC40u060aUyNVkTdCA7ApjHgFvqGsiTiHS2CcG
    mqBt81wHgs9oSkA1tHnMJToiFtLOX6/agYp1eZEH9BtYBXQXAsqvwP3bsw9J/nL/
    +nwCN658YpyX8G5Y3t9AhvGZxMro2Bqhaxv1uaFQDKM5oZSstIZ8HhLieqgft6AR
    aUGF1tgRk8qqClNdEYWZIGC98wzl7BcnBqezpt3jQVTky6SIb+hbfYOeLFayvEkb
    mNeQ2yQqMSU29aelxPXZd4j2tpXj79GdC8tWrSC8xWThSlE5EWFiG+RHMfuLla63
    n4ixrkwvmOg4hnM2MFWJ1vHRG7Kysw8Axa5qd/pe878kXPKQZYjZyybj2Kue4Pc8
    OTYA2AzkI/oqIb41znY1/a8F06UHGfzSgCLlNP8i8iDd1rPBrqzaEQQy0f3LtFPb
    vnCtBF5wDGP4wmRW8K5Js/1W+pkA59lRh/xkTUmjLXvH11tUbSw/FSGx4EMGW2dq
    p2QupviHmuKQ8sqjJdvRlnECAwEAAQ==
    -----END PUBLIC KEY-----", "until": "2021-03-04T05:06:29Z"}
