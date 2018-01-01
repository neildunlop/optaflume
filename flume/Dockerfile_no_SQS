FROM probablyfine/flume

#copy the local flume.conf file to the docker image
ADD flume.conf /var/tmp/flume.conf

#expose a port so that we can connect to it using netcat to provide input
EXPOSE 44444
EXPOSE 80

#provide some entry point command line parameters so that flume starts and reads the right config
ENTRYPOINT [ "flume-ng", "agent", "-c", "/opt/flume/conf", "-f", "/var/tmp/flume.conf", "-n", "docker", "-Dflume.root.logger=INFO,console" ]
