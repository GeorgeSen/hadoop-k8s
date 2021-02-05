FROM       jinbozi/hdfs:0.6
MAINTAINER nwpu

COPY start.sh /custom
COPY core-site.xml /custom/hadoop-2.9.0/etc/hadoop
COPY hdfs-site.xml /custom/hadoop-2.9.0/etc/hadoop
COPY yarn-site.xml /custom/hadoop-2.9.0/etc/hadoop
COPY mapred-site.xml /custom/hadoop-2.9.0/etc/hadoop
RUN rm /custom/prometheus_export/*java
RUN rm /custom/prometheus_export/*class
RUN rm /custom/prometheus_export/*md
RUN apt-get install -y sysstat
COPY get_system_metrics.sh /custom/prometheus_export
COPY HDFSDiskExport.java /custom/prometheus_export
COPY HDFSDiskExport.class /custom/prometheus_export
COPY HDFSDiskCollector.class /custom/prometheus_export

# Hdfs ports
EXPOSE 9000 50010 50020 50070 50075 50090
# Mapred ports
EXPOSE 19888
# Yarn ports
EXPOSE 8030 8031 8032 8033 8040 8042 8088
# Other ports
EXPOSE 49707 31822
# Prometheus port
EXPOSE 10318

RUN chmod 777 /custom/start.sh
RUN chmod 777 /custom/prometheus_export/get_system_metrics.sh
CMD ["/custom/start.sh"]
