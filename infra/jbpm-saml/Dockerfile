FROM quay.io/kiegroup/jbpm-server-full:latest

USER root
ENV JBOSS_HOME=/opt/jboss/wildfly
WORKDIR $JBOSS_HOME

# Tools we need
RUN microdnf install -y unzip curl && microdnf clean all

# --- Install Keycloak SAML adapter for WildFly (manual, robust) ---
# Pick a Keycloak version compatible with your Keycloak server.
# 26.0.5 shown as example; change if you run a different Keycloak version.
ENV KC_VERSION=26.0.5
RUN set -e; \
  curl -fsSL -o /tmp/keycloak-saml-adapter.zip \
    https://github.com/keycloak/keycloak/releases/download/${KC_VERSION}/keycloak-saml-adapter-dist-${KC_VERSION}.zip ; \
  unzip -q /tmp/keycloak-saml-adapter.zip -d /tmp/kc ; \
  $JBOSS_HOME/bin/jboss-cli.sh --file=/tmp/kc/keycloak-saml-adapter-dist-${KC_VERSION}/wildfly-install-saml-adapter.cli ; \
  rm -rf /tmp/kc /tmp/keycloak-saml-adapter.zip

# --- Ensure the jBPM webapps are exploded so we can drop per-WAR configs ---
# If they are already directories, the conditions will be skipped.
RUN set -e; \
  cd $JBOSS_HOME/standalone/deployments; \
  if [ -f business-central.war ]; then mkdir -p business-central.war.exploded && unzip -q business-central.war -d business-central.war.exploded && rm -f business-central.war && mv business-central.war.exploded business-central.war; fi; \
  if [ -f kie-server.war ]; then mkdir -p kie-server.war.exploded && unzip -q kie-server.war -d kie-server.war.exploded && rm -f kie-server.war && mv kie-server.war.exploded kie-server.war; fi

# --- Copy SAML config into each WAR ---
# Provide these two files next to this Dockerfile.
COPY business-central-keycloak-saml.xml $JBOSS_HOME/standalone/deployments/business-central.war/WEB-INF/keycloak-saml.xml
COPY kie-server-keycloak-saml.xml      $JBOSS_HOME/standalone/deployments/kie-server.war/WEB-INF/keycloak-saml.xml

# Match the base image user (WildFly runs as UID 185)
USER 185

