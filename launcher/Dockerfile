# -buster is required to have apt available
FROM openjdk:17-slim-buster

# Optional JVM arguments, such as memory settings
ARG JVM_ARGS=""
ARG APPINSIGHTS_AGENT_VERSION=3.2.11

# Install curl, then delete apt indexes to save image space
RUN apt update \
    && apt install -y curl \
    && rm -rf /var/cache/apt/archives /var/lib/apt/lists

WORKDIR /app

RUN curl --fail -LO https://github.com/microsoft/ApplicationInsights-Java/releases/download/$APPINSIGHTS_AGENT_VERSION/applicationinsights-agent-$APPINSIGHTS_AGENT_VERSION.jar

COPY ./build/libs/app.jar /app

EXPOSE 8181
EXPOSE 9191
EXPOSE 8282

# health status is determined by the availability of the /health endpoint
HEALTHCHECK --interval=5s --timeout=5s --retries=10 CMD curl -H "X-Api-Key: $EDC_API_AUTH_KEY" --fail http://localhost:8181/api/check/health

ENV WEB_HTTP_PORT="8181"
ENV WEB_HTTP_PATH="/api"
ENV WEB_HTTP_DATA_PORT="9191"
ENV WEB_HTTP_DATA_PATH="/api/v1/data"
ENV WEB_HTTP_IDS_PORT="8282"
ENV WEB_HTTP_IDS_PATH="/api/v1/ids"

# Use "exec" for graceful termination (SIGINT) to reach JVM.
# ARG can not be used in ENTRYPOINT so storing values in ENV variables
ENV JVM_ARGS=$JVM_ARGS
ENV APPINSIGHTS_AGENT_VERSION=$APPINSIGHTS_AGENT_VERSION
ENTRYPOINT [ "sh", "-c", \
    "exec java -javaagent:applicationinsights-agent-$APPINSIGHTS_AGENT_VERSION.jar $JVM_ARGS -jar app.jar"]
