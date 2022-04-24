Configurations directory for Docker Registry and Docker Auth.

The following documentation assumes that the root path for this whole thing is /opt/docker-registry.
Feel free to screw the thing up by changing it.
Remember to change, however, the example.domain.com with whatever you want it to be.

If you're using Let's Encrypt, as in this example, first you might need to do:

- cp fullchain.pem fullchain.crt
- cp privkey.pem domain.key
- cat cert.pem chain.pem > domain.crt

The first two for permissions reasons, I believe, and the last one because it's ideal to have the fullchain in a single thing. 

Locations:

- auth/config/auth_config.yml - That's where the authentication provider is configured. References about what can be customized here can be found in: https://github.com/cesanta/docker_auth/blob/main/examples/reference.yml
- config/config.yml - That's where the registry itself is configured. References can be found here: https://docs.docker.com/registry/configuration/


Notes: 

- The value of "token:issuer" in auth/config/auth_config.yml has to match the value of "auth:token:realm" in config/config.yml, otherwise it craps itself.
- The passwords for the users specified in auth/config/auth_config.yml need to be encrypted with Blowfish, I think? (can be generated with htpasswd -B)


Launch commands:

Docker Auth:
-------------

docker run \
    --restart=always --name docker_auth -p 5001:5001 \
    -v /etc/letsencrypt/live/example.domain.com:/certs \
    -v /opt/docker-registry/auth/config:/config:ro \
    -v /opt/docker-registry/auth/logs:/logs \
    cesanta/docker_auth:1 /config/auth_config.yml


Docker Registry:
-----------------

docker run -d -p 443:5000 --restart=always --name registry \
  -v /opt/docker-registry/config/config.yml:/etc/docker/registry/config.yml \
  -v /etc/letsencrypt/live/example.domain.com:/certs \
  -v /opt/docker-registry/registry:/var/lib/registry \
  -v /opt/docker-registry/auth:/auth \
  registry:2


Good luck with that.

