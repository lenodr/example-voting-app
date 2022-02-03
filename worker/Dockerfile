FROM maven:3.8.4-jdk-8

WORKDIR /app 

COPY . . 

RUN mvn package -DskipTests && \ 
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf /app/*

CMD java -jar /run/worker.jar 
