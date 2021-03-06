# Prosody server with a OTRv4 prekey server included

A docker composed infrastructure containing a [prosody](https://prosody.im/) xmpp server and a [OTRv4](https://github.com/otrv4/libotr-ng) prekey server added to it. 

It also has some [dummy accounts](#available-accounts).

__IMPORTANT__: This server is intended to be used only for **local testing**.

## Content

- [Running the xmpp server on Docker](#running-the-xmpp-server-on-docker)
- [Restarting the prosody server](#restarting-the-prosody-server) 
- [Available accounts](#available-accounts)
- [Troubleshoots](#troubleshoots)

## Running the xmpp server on Docker

1. Create an `.env` file that looks exactly like this:

```
  PREKEY_SERVER_IDENTITY=prekeys.localhost
  PREKEY_SERVER_FINGERPRINT=
  XMPP_COMPONENTS_SECRET=this is secret
```

2. Run:

```
  $ docker-compose up
```

3. Copy the shown fingerprint (without spaces/brackets) into the `.env` file. Something like this will be shown:

```
prekeys-raw_1   | Starting server on 0.0.0.0:30123...
prekeys-raw_1   | [F91453A3D80BB48F FD683C98FBB2ED30 33F52861F6549F64 A45216627D43BA80 CCD9B08156CBAB97 E9B00314B313AF87 6FD6FBAFE97C00CC]
prekeys-xmpp_1  | fingerprint provided is not valid
```

This is line shows you the appropriate fingerprint 

`prekeys-raw_1   | [F91453A3D80BB48F FD683C98FBB2ED30 33F52861F6549F64 A45216627D43BA80 CCD9B08156CBAB97 E9B00314B313AF87 6FD6FBAFE97C00CC]`.

Copy it on the `.env` file, so it looks like this:

```
PREKEY_SERVER_IDENTITY=prekeys.localhost
PREKEY_SERVER_FINGERPRINT=F91453A3D80BB48FFD683C98FBB2ED3033F52861F6549F64A45216627D43BA80CCD9B08156CBAB97E9B00314B313AF876FD6FBAFE97C00CC
XMPP_COMPONENTS_SECRET=this is secret
```

4. Run:

```
$ docker-compose up
```

Everything should run. This line should show up:

```
xmpp-server_1   | prekeys.localhost:component  info	External component successfully authenticated
```

In order to make this server be recognizable by the [pidgin plugin](https://github.com/otrv4/pidgin-otrng), you need to run the following command:

```
  ./configure CFLAGS="-ggdb3 -O0 -DDEFAULT_PREKEYS_SERVER='\"prekeys.localhost\"'"
```

## Restarting the prosody server

Run:

```
$ docker-compose down
$ docker-compose stop
$ docker system prune -a
```

## Available accounts

```
+-------------------------------------+
|   ACCOUNT            |   PASSWORD   |
+-------------------------------------+
|   alice@localhost    |   alice      |
+-------------------------------------+
|   bob@localhost      |   bob        |
+-------------------------------------+
```

## Troubleshoots

If you need to fix [ownership]([https://prosody.im/doc/certificates](https://prosody.im/doc/certificates)) issues using `prosody`, you can do the following:

1. In another terminal window run:

```
$ docker-compose exec xmpp-server bash
$ ls -al ~/localhost/
```

2. Take the `id/gid` (the 4th column) of the offline folder. `chown` it outside of the docker container so the offline storage is writable. 

Note that the user must belong to the docker group.

```
$ sudo chown -R uid:gid prekey-server-docker-compose/prosody/data/localhost/
```

3. In some distributions could be necessary to execute the following steps:

- Modify `prekey-server-docker-compose/prosody/Dockerfile` adding:

```
$ chown root:prosody /etc/prosody/certs/localhost.key
$ chmod 660 /etc/prosody/certs/localhost.key
```

- Rebuild the image with:

```
$ sudo docker-compose build
```
