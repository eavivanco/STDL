# Séance 6

Here you will find the folder `comp_socle` which contains the files of the container.

## Part 1

1. Installez et exécutez SonarQube dans un conteneur Docker https://docs.sonarqube.org/latest/setup/get-started-2-minutes/
- The file `SonarQube_running.png` contains the screenshot running SonarQube on Docker

2. Analysez et expliquez ce que vous comprenez de la définition du Dockerfile de SonarQube (9/community)
   
- `FROM alpine:3.14` : Alpine is a Image base
  
- `ENV LANG='en_US.UTF-8' \` : Set the languages supported by the container
    `LANGUAGE='en_US:en' \`
    `LC_ALL='en_US.UTF-8'`

- `ARG SONARQUBE_VERSION=9.2.2.50622` : set the SonarQube version
- `ARG SONARQUBE_ZIP_URL=https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-${SONARQUBE_VERSION}.zip` : set the directory from where we are gonna obtain the SonarQube version specified before
- `ENV JAVA_HOME='/usr/lib/jvm/java-11-openjdk' \` : we establish the java environment with its openJDK version
    `PATH="/opt/java/openjdk/bin:$PATH" \` : we specify the path from where we gonna obtain the openJDK
    `SONARQUBE_HOME=/opt/sonarqube \` : we establish the SonarQube directory inside the environment
    `SONAR_VERSION="${SONARQUBE_VERSION}" \` we specify the SonarQube version
    `SQ_DATA_DIR="/opt/sonarqube/data" \` : we establish the SonarQube data directory inside the environment
    `SQ_EXTENSIONS_DIR="/opt/sonarqube/extensions" \` : we establish the SonarQube extensions directory inside the environment
    `SQ_LOGS_DIR="/opt/sonarqube/logs" \` : we establish the SonarQube logs directory inside the environment
    `SQ_TEMP_DIR="/opt/sonarqube/temp"` : we establish the SonarQube temporary files directory inside the environment
- `RUN set -eux; \` : run the container
    `addgroup -S -g 1000 sonarqube; \` : add the group of SonarQubre called "sonarqube"
    `adduser -S -D -u 1000 -G sonarqube sonarqube; \` : add an user to the group "sonarqube" also called "sonarqube"
    `apk add --no-cache --virtual build-dependencies gnupg unzip curl; \` : Alpine Linux package manager adding the package "build-dependencies" "gnupg" "unzip" "curl"
    `apk add --no-cache bash su-exec ttf-dejavu openjdk11-jre; \` : Alpine Linux package manager adding the package "su-exec" "gnupg" "ttf-dejavu" "openjdk11-jre" using bash
    `echo "networkaddress.cache.ttl=5" >> "${JAVA_HOME}/conf/security/java.security"; \` : display "networkaddress.cache.ttl=5" and append to it "${JAVA_HOME}/conf/security/java.security"
    `sed --in-place --expression="s?securerandom.source=file:/dev/random?securerandom.source=file:/dev/urandom?g" "${JAVA_HOME}/conf/security/java.security"; \`
    `for server in $(shuf -e ha.pool.sks-keyservers.net \`
    `                        hkp://p80.pool.sks-keyservers.net:80 \`
    `                        keyserver.ubuntu.com \`
    `                        hkp://keyserver.ubuntu.com:80 \`
    `                        pgp.mit.edu) ; do \`
    `    gpg --batch --keyserver "${server}" --recv-keys 679F1EE92B19609DE816FDE81DB198F93525EC1A && break || : ; \`
    `done; \`
    `mkdir --parents /opt; \` : make a directory called "opt"
    `cd /opt; \` : acces to "opt"
    `curl --fail --location --output sonarqube.zip --silent --show-error "${SONARQUBE_ZIP_URL}"; \` : if fails acces to the error
    `curl --fail --location --output sonarqube.zip.asc --silent --show-error "${SONARQUBE_ZIP_URL}.asc"; \` : if fails acces to the error
    `gpg --batch --verify sonarqube.zip.asc sonarqube.zip; \` :
    `unzip -q sonarqube.zip; \` : unzips the "sonarqube.zip" compressed file
    `mv "sonarqube-${SONARQUBE_VERSION}" sonarqube; \` : move the file "sonarqube-${SONARQUBE_VERSION}" to sonarqube
    `rm sonarqube.zip*; \` : remove the compressed file "sonarqube.zip"
    `rm -rf ${SONARQUBE_HOME}/bin/*; \` : remove the reference path "${SONARQUBE_HOME}/bin/*"
    `chown -R sonarqube:sonarqube ${SONARQUBE_HOME}; \` : assign sonarqube as the owner and sonarqube es the group of "${SONARQUBE_HOME}"
    `chmod -R 777 "${SQ_DATA_DIR}" "${SQ_EXTENSIONS_DIR}" "${SQ_LOGS_DIR}" "${SQ_TEMP_DIR}"; \` : allows every user to read/write/execute over `"${SQ_DATA_DIR}"`, `"${SQ_EXTENSIONS_DIR}"`, `"${SQ_LOGS_DIR}"`, `"${SQ_TEMP_DIR}"` roots
    `apk del --purge build-dependencies;` : Alpine Linux package manager deleting "build-dependencies"
- `COPY --chown=sonarqube:sonarqube run.sh sonar.sh ${SONARQUBE_HOME}/bin/`
- `WORKDIR ${SONARQUBE_HOME}` : set "${SONARQUBE_HOME}" as working directory
- `EXPOSE 9000` : informs Docker that the container listens on the 9000 network port at runtime (does not make the port)
- `STOPSIGNAL SIGINT` : stop signal (the program stops) setted as `CTRL + C`
- `ENTRYPOINT ["/opt/sonarqube/bin/run.sh"]` : we configure that the container will be executed if we run "/opt/sonarqube/bin/run.sh"
- `CMD ["/opt/sonarqube/bin/sonar.sh"]` : we provide "/opt/sonarqube/bin/sonar.sh" as default for executing the container

## Part 2

Construire une image qui permet d’utiliser `Maven 3.8.3` avec `Open JDK 11`:

1. L’emplacement du projet Maven est précisé au moment de l’instanciation du conteneur
2. L’emplacement du dépôt local Maven est également précisé au moment de l’instanciation du conteneur

- You will find the code screenshot on `MavenImplementatio.png`

3. Les commandes Maven peuvent être saisies à l’aide d’un shell ouvert dans le conteneur et arrivant à la racine de votre projet Java (emplacement du fichier pom.xml)


### Using Maven with the Maven Docker Image

- `$ docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-8 mvn clean install`

### Building your own Docker Maven Image (base)
- `$ docker build --tag my_local_maven:3.5.2-jdk-8`

### Using Docker the Maven local repository (base)
- `$ docker volume create --name maven-repo`
- `$ docker run -it -v maven-repo:/root/.m2 maven mvn archetype:generate # will download artifacts`

### To pack a local Maven repository with the Docker image we have to add the following to the `Dockerfile`

- `COPY pom.xml /tmp/pom.xml`
- `RUN mvn -B -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve`