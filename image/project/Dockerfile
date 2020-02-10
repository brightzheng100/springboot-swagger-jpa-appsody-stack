#### GLOBAL
ARG PROJECT_DIR="/project"


#### COMIPLE STAGE
FROM {{.stack.baseimage}} as compile

# Args
ARG PROJECT_DIR

# Ensure up to date / patched OS
RUN  apt-get -qq update \
  && apt-get -qq install -y curl wget xmlstarlet unzip \
  && DEBIAN_FRONTEND=noninteractive apt-get -qq upgrade -y \
  && apt-get -qq clean \
  && rm -rf /tmp/* /var/lib/apt/lists/*

# Create non-root user / group to run
RUN  groupadd --gid 1000 java_group \
  && useradd --uid 1000 --gid java_group --shell /bin/bash --create-home java_user \
  && mkdir -p /mvn/repository && chown -R java_user:java_group /mvn \
  && mkdir ${PROJECT_DIR} && chown -R java_user:java_group ${PROJECT_DIR}

# Switch to non-root user and workdir
USER java_user:java_group
WORKDIR ${PROJECT_DIR}

# Layer caching for stack dependencies
LABEL "stack.appsody.dev/id"="{{.stack.image.namespace}}/{{.stack.id}}"
LABEL "stack.appsody.dev/version"="{{.stack.version}}"
COPY --chown=java_user:java_group parent-pom.xml /mvn/parent-pom.xml
RUN mvn --no-transfer-progress -B dependency:go-offline install -Dmaven.test.skip=true -f /mvn/parent-pom.xml

# Layer caching for application dependencies
COPY --chown=java_user:java_group user-app/pom.xml /mvn/pom.xml
RUN mvn --no-transfer-progress -B dependency:go-offline -f /mvn/pom.xml

# Copy and build project, .dockerignore should apply here
COPY --chown=java_user:java_group . ./

# Switch to user-app as workdir
WORKDIR ${PROJECT_DIR}/user-app

# Run Maven
RUN /project/util/check_version build \
 && /project/build.sh package \
 && /project/build.sh createStartScript


#### BUILD STAGE
FROM {{.stack.basebuildimage}}

# Ensure up to date / patched OS
RUN  apt-get -qq update \
  && DEBIAN_FRONTEND=noninteractive apt-get -qq upgrade -y \
  && apt-get -qq clean \
  && rm -rf /tmp/* /var/lib/apt/lists/*

# Create non-root user / group to run
RUN groupadd --gid 1000 java_group \
  && useradd --uid 1000 --gid java_group --shell /bin/bash --create-home java_user

# Args
ARG ARTIFACT_TITLE=springboot-swagger-jpa-stack
ARG ARTIFACT_VERSION=1.0.0-SNAPSHOT
ARG DEPENDENCY=/project/user-app/target/dependency

# Envs
ENV JVM_ARGS=""

# Lable the image
LABEL org.opencontainers.image.title="${ARTIFACT_TITLE}"
LABEL org.opencontainers.image.version="${ARTIFACT_VERSION}"

# Copy files
COPY --chown=java_user:java_group --from=compile /project/user-app/target/start.sh /app/start.sh
COPY --chown=java_user:java_group --from=compile ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --chown=java_user:java_group --from=compile ${DEPENDENCY}/META-INF /app/META-INF
COPY --chown=java_user:java_group --from=compile ${DEPENDENCY}/BOOT-INF/classes /app

# Switch to non-root user and workdir
USER java_user:java_group

# Expose ports
EXPOSE 8080
EXPOSE 8443

# Set the entrypoint
ENTRYPOINT [ "/app/start.sh" ]
