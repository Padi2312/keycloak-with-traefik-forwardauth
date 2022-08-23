# Keycloak with Traefik and ForwardAuth 

Simple setup for using Keycloak and Traefik. 

In addition to that i provide an example of using keycloak for securing a service running behind traefik.

# Info
The middleware being used to forward auth is defined in `./config/dynamic.yml` with `keycloak` as name.

# Getting started

***Note: For Docker Compose Version 1 use `docker-compose` instead of `docker compose`***

### Start traefik
1. Setup `DOMAIN` in `.env` 
2. Enter you email in `./config/traefik.yml` in line 30 at `example@mail.invalid` 
3. Modify file rights of `./config/acme.json` to 600
   - `chmod 600 ./config/acme.json`
4. Create an external network `traefik`
   - `docker network create traefik`
5. `docker compose up -d traefik`

### Start keycloak
5. `docker compose up -d keycloak`
6. Setup a realm with a client and a user 
   - For more information how to setup a realm please see: https://www.keycloak.org/getting-started/getting-started-docker
   - **Note:** The access type of client have to be `confidential` to get a client secrect
   - ### **IMPORTANT: The user must have a email address otherwise the forwardauth will not work**
7. In client settings you have to set `Valid Redirect URIs` with the url of your service
   - *Example:* https://yourdomain.com/*
   - It's important to add asterix symbol at the end
8. Enter `CLIENT_ID`,`CLIENT_SECRET` and `REALM` in `.env`

### Start forwardauth container
9. *Optional* Change `SECRET` in `docker-compose.yml` under forwardauth environment 
10. `docker compose up -d forwardauth`

### Secure a service with keycloak
11.  Add `keycloak@file` as middleware to the service\
    -  Example: `- "traefik.http.routers.whoami.middlewares=keycloak@file,secHeaders@file"` \
    - **Note:** The middleware is already added to our example service
12. `docker-compose up -d`


# Troubleshooting

## Errors
If you encounter any errors feel free to contact me.

## Bad Gateway 
If you got a "bad gateway" error you will receive a URL like this: 

https://example.yourdomain.com/_oauth?error=invalid_scope&error_description=Invalid+scopes%3A+openid+profile+email+groups&state= 

To fix this error you have to assign the correct client scopes to your client. In our case you first have to create two scopes not being given by default.  \
Create the `openid` and `groups` scope (Settings can be default).
Next asign scopes `openid`, `groups`,`profile` and `email` to the used client.

## Not authorized
Check whether you have set the email for your user and try again