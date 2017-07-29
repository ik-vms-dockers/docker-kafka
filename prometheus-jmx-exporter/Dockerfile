FROM solsson/kafka-jre

ENV EXPORTER_VERSION=e4d41d264363d07debf900ae1f420531214e437d
ENV EXPORTER_REPO=github.com/prometheus/jmx_exporter

WORKDIR /usr/local/

RUN set -ex; \
  export DEBIAN_FRONTEND=noninteractive; \
  runDeps=''; \
  buildDeps='curl ca-certificates maven'; \
  apt-get update && apt-get install -y $runDeps $buildDeps --no-install-recommends; \
  \
  mkdir ./jmx_exporter; \
  curl -SLs https://$EXPORTER_REPO/archive/$EXPORTER_VERSION.tar.gz | tar -xzf - --strip-components=1 -C ./jmx_exporter; \
  cd ./jmx_exporter; \
  mvn package; \
  find jmx_prometheus_httpserver/ -name *-jar-with-dependencies.jar -exec mv -v '{}' ../jmx_prometheus_httpserver.jar \;; \
  mv example_configs ../; \
  cd ..; \
  \
  rm -Rf ./jmx_exporter /root/.m2; \
  \
  apt-get purge -y --auto-remove $buildDeps; \
  rm -rf /var/lib/apt/lists/*; \
  rm -rf /var/log/dpkg.log /var/log/alternatives.log /var/log/apt

# Use a sample config that also has a Grafana dashboard https://blog.rntech.co.uk/2016/10/20/monitoring-apache-kafka-with-prometheus/
# Mount your own yml, for example using ConfigMap, or set Kafka JMX_PORT=5555
RUN set -ex; \
  buildDeps='curl ca-certificates'; \
  apt-get update && apt-get install -y $buildDeps --no-install-recommends; \
  curl -SLs -o example_configs/kafka-prometheus-monitoring.yml \
    https://raw.githubusercontent.com/rama-nallamilli/kafka-prometheus-monitoring/c4ee0e6b03386375b9b9e66b3fcbf4a704bec8f5/prometheus-jmx-exporter/confd/templates/kafka.yml.tmpl; \
  sed -i 's|{{ getv "/jmx/host" }}|127.0.0.1|' example_configs/kafka-prometheus-monitoring.yml; \
  sed -i 's|{{ getv "/jmx/port" }}|5555|'      example_configs/kafka-prometheus-monitoring.yml; \
  apt-get purge -y --auto-remove $buildDeps; \
  rm -rf /var/lib/apt/lists/*; \
  rm -rf /var/log/dpkg.log /var/log/apt

ENTRYPOINT ["java", "-jar", "jmx_prometheus_httpserver.jar"]
CMD ["5556", "example_configs/kafka-prometheus-monitoring.yml"]