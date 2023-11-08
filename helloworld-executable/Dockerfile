FROM docker.io/ibmjava:jre

RUN mkdir /opt/app

COPY target/*.jar /opt/app/japp.jar

CMD ["java", "-jar", "/opt/app/japp.jar"]