####
# This Dockerfile is used in order to build a container that runs the Spring Boot application
#
# Build the image with:
#
# docker build -f docker/Dockerfile -t springboot/sample-demo .
#
# Then run the container using:
#
# docker run -i --rm -p 8081:8081 springboot/sample-demo
# bellsoft/liberica-openjdk-alpine-musl:17
####
FROM arm64v8/maven

WORKDIR /build

# Build dependency offline to streamline build
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src src
RUN mvn package -Dmaven.test.skip=true 
# compute the created jar name and put it in a known location to copy to the next layer.
# If the user changes pom.xml to have a different version, or artifactId, this will find the jar 
RUN grep version /build/target/maven-archiver/pom.properties | cut -d '=' -f2 >.env-version 
RUN grep artifactId /build/target/maven-archiver/pom.properties | cut -d '=' -f2 >.env-id
RUN mv /build/target/$(cat .env-id)-$(cat .env-version).jar /build/target/export-run-artifact.jar
RUN java -Djarmode=layertools -jar /build/target/export-run-artifact.jar extract

FROM openjdk:11-jdk
COPY --from=0 build/dependencies/ ./
COPY --from=0 build/snapshot-dependencies/ ./
COPY --from=0 build/spring-boot-loader/ ./
COPY --from=0 build/application/ ./

EXPOSE 8081
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher", "--server.port=8081"]
