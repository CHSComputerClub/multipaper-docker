#syntax=docker/dockerfile:1.4

# Create a shared base container
FROM eclipse-temurin:22-jre as base

RUN useradd -d /app -s /bin/false multipaper
USER multipaper
WORKDIR /app


# Build the MultiPaper-Master container
FROM base as master

ARG RELEASE_URL
ADD --chown=multipaper:multipaper $RELEASE_URL /opt/multipaper-master/multipaper-master.jar

ENTRYPOINT [ "java", "-jar", "/opt/multipaper-master/multipaper-master.jar" ]
CMD [ "35353", "25565" ]

EXPOSE 35353/tcp 25565/tcp
VOLUME [ "/app" ]


# Build the Multipaper server container
FROM base as server

# Create an entry script based on akiicat/MultiPaper-Container
COPY --chown=multipaper:multipaper --chmod=744 <<-"EOT" /opt/multipaper/entry.sh
#!/usr/bin/env bash
# Grab files and folders seperated by a semicolon from the environment variable $MASTERFILES and put them into /app
if [[ -n "$MASTERFILES" ]]; then
    echo "Copying master files..."
    IFS=';' read -ra FILES <<< "$MASTERFILES"
    for FILE in "${FILES[@]}"; do
        # split the file path into the directory and the file name
        DIR=$(dirname "$FILE")
        FILENAME=$(basename "$FILE")
        # if dir is just ., then run base cp, otherwise create the directory and copy the file with rel path
        if [[ "$DIR" == "." ]]; then
            cp -r "/masterData/$FILENAME" "/app"
        else
            mkdir -p "/app/$DIR"
            cp -r "/masterData/$FILENAME" "/app/$DIR"
        fi
    done
fi

# If the EULA environment variable is true, accept it
if [[ -n "$EULA" ]]; then
    echo "Accepting the EULA..."
    echo "eula=$EULA" > eula.txt
fi
# Run the MultiPaper Server jar file
java -jar $@
EOT

ARG RELEASE_URL
ADD --chown=multipaper:multipaper $RELEASE_URL /opt/multipaper/multipaper.jar

ENTRYPOINT [ "/opt/multipaper/entry.sh", "/opt/multipaper/multipaper.jar" ]

EXPOSE 25565/tcp
VOLUME [ "/app" ]
