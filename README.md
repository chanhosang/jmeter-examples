
- [How to run JMeter Test with Taurus?](#how-to-run-jmeter-test-with-taurus)
- [How to run JMeter Test with Maven?](#how-to-run-jmeter-test-with-maven)
- [How to run JMeter Test in Docker Container?](#how-to-run-jmeter-test-in-docker-container)
## How to run JMeter Test with Taurus?

**Pre-requisite**
* Any OS
* Installed [Taurus](https://gettaurus.org/install/Installation/)

To run load test with default settings:
```
bzt scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml
```

To run load test with default settings, and monitor life stats from BlazeMeter Service Dashboard:
```
bzt scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml -report
```

To run load test by overriding the user defined variables and thread group properties via command line:
```
bzt scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml \
-o execution.0.concurrency=1 \
-o execution.0.ramp-up=1s \
-o execution.0.iterations=2 \
-o execution.1.concurrency=1 \
-o execution.1.ramp-up=1s \
-o execution.1.hold-for=1m \
-report
```

To run load test and generate JMeter HTML Report upon test completion:
```
bzt scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml scripts/bzt/bzt-jmeter-load-test-multiple-scenarios-reporting.yml\
-o modules.jmeter.properties="{'jmeter.reportgenerator.overall_granularity':60000}"
-o modules.jmeter.properties="{'jmeter.reportgenerator.report_title': Load Test Dashboard}"
```

To know more about Taurus, refer to:
* [Taurus Command Line](https://gettaurus.org/docs/CommandLine/)
* [Taurus YAML Tutorial](https://gettaurus.org/docs/YAMLTutorial/)
* [Taurus Configuration Syntax](https://gettaurus.org/docs/ConfigSyntax/)


## How to run JMeter Test with Maven?

**Pre-requisite**
* Any OS
* Installed [Maven](https://maven.apache.org/install.html)

To run load test with default settings:
```
mvn clean verify -P jmeter_load_test_purchase_product -f jmeter/pom.xml
```

To run load test by overriding the user defined variables via command line:
```
mvn clean verify -P jmeter_load_test_purchase_product -f jmeter/pom.xml \
-DnumberOfThreads=1 -DloopCount=2 -DrampUp=1
```


To know more about Maven, refer to:
* [Maven in 5 minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
* [Maven Commands Options Cheat Sheet](https://www.journaldev.com/33645/maven-commands-options-cheat-sheet)
* [POM Reference](https://maven.apache.org/pom.html)



## How to run JMeter Test in Docker Container?

**Pre-requisite**
* Installed [Docker](https://docs.docker.com/get-docker/)


If you don't want to install *Maven* or *Taurus* in your machine, you can actually run the test inside docker container.

To run load test with **Taurus** inside [blazemeter/taurus docker container](https://hub.docker.com/r/blazemeter/taurus):
```
docker run --rm \
-v $HOME/.bzt:/root/.bzt \
-v $HOME/performance-test-demo:/bzt-configs \
-v $HOME/performance-test-demo/results:/tmp/artifacts \
blazemeter/taurus \
scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml
```

To run load test with **Maven** inside [maven docker container](https://hub.docker.com/_/maven):
```
docker run --rm \
-v $HOME/.m2:/root/.m2 \
-v $HOME/performance-test-demo:/opt/maven \
-w /opt/maven \
maven:3-jdk-8 \
mvn clean verify -P jmeter_load_test -f jmeter/pom.xml
```

To know more about docker command, refer to:
* [Docker Cheat Sheet](https://www.docker.com/sites/default/files/d8/2019-09/docker-cheat-sheet.pdf)
