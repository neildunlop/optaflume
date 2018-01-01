FROM probablyfine/flume

#copy the local flume.conf file to the docker image
ADD flume.conf /var/tmp/flume.conf

#make the SQS sink plugin target directory
RUN mkdir -p /opt/flume/plugins.d/flume-sqs-source/lib

#make the SQS sink plugin depedency target directory
RUN mkdir -p /opt/flume/plugins.d/flume-sqs-source/libext

#copy the SQS Sink to the docker image
ADD flume-sqs-source-1.1.0.jar opt/flume/plugins.d/flume-sqs-source/lib/flume-sqs-source-1.1.0.jar

#copy the SQS Sink to the docker image
ADD aws-java-sdk-core-1.9.6.jar aws-java-sdk-sqs-1.9.6.jar aws-java-sdk-sts-1.9.6.jar /opt/flume/plugins.d/flume-sqs-source/libext/

#expose a port so that we can connect to it using netcat to provide input
EXPOSE 44444
EXPOSE 80

#provide some entry point command line parameters so that flume starts and reads the right config
ENTRYPOINT [ "flume-ng", "agent", "-c", "/opt/flume/conf", "-f", "/var/tmp/flume.conf", "-n", "docker", "-Dflume.root.logger=INFO,console" ]
