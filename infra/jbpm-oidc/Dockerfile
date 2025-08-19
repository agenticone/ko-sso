# No fetch stage needed anymore
FROM quay.io/kiegroup/jbpm-server-full:latest

ENV JBOSS_HOME=/opt/jboss/wildfly
WORKDIR $JBOSS_HOME

# --- Explode WARs so we can drop config files ---
RUN set -e; cd $JBOSS_HOME/standalone/deployments; \
  if [ -f business-central.war ]; then \
    mkdir -p bc && (cd bc && jar -xf ../business-central.war) && rm -f business-central.war && mv bc business-central.war; \
  fi; \
  if [ -f kie-server.war ]; then \
    mkdir -p ks && (cd ks && jar -xf ../kie-server.war) && rm -f kie-server.war && mv ks kie-server.war; \
  fi

# --- Provide per-deployment OIDC config (picked up by Elytron OIDC client) ---
# Create WEB-INF/oidc.json for Business Central
ARG KC_REALM=jbpm
ARG KC_URL=https://keycloak.local.test/
ARG KC_CLIENT_ID=jbpm-business-central
ARG KC_CLIENT_SECRET=CHANGE-ME
RUN set -e; cat > $JBOSS_HOME/standalone/deployments/business-central.war/WEB-INF/oidc.json <<JSON
{
  "realm": "${KC_REALM}",
  "auth-server-url": "${KC_URL}",
  "ssl-required": "external",
  "resource": "${KC_CLIENT_ID}",
  "credentials": { "secret": "${KC_CLIENT_SECRET}" },
  "principal-attribute": "email",
  "confidential-port": 0,
  "bearer-only": false
}
JSON

# --- Enable Elytron OIDC subsystem & secure the deployment ---
# This adds the extension + subsystem and a secure-deployment entry for Business Central.
# (Equivalent to configuring it in standalone.xml.)
RUN cat > /tmp/enable-elytron-oidc.cli <<'CLI'
if (outcome != success) of /extension=org.wildfly.extension.elytron-oidc-client:read-resource
  /extension=org.wildfly.extension.elytron-oidc-client:add
end-if
if (outcome != success) of /subsystem=elytron-oidc-client:read-resource
  /subsystem=elytron-oidc-client:add
end-if
if (outcome != success) of /subsystem=elytron-oidc-client/secure-deployment=business-central.war:read-resource
  /subsystem=elytron-oidc-client/secure-deployment=business-central.war:add( \
    client-id="${KC_CLIENT_ID}", \
    provider-url="${KC_URL}realms/${KC_REALM}", \
    principal-claim="email", \
    ssl-required="EXTERNAL", \
    confidential=true)
  /subsystem=elytron-oidc-client/secure-deployment=business-central.war/credential=secret:add(secret="${KC_CLIENT_SECRET}")
end-if
:reload
CLI
RUN $JBOSS_HOME/bin/jboss-cli.sh --file=/tmp/enable-elytron-oidc.cli && rm -f /tmp/enable-elytron-oidc.cli

# (Optional) Repeat the same pattern for kie-server.war if you want it OIDC-protected too.

USER 185

