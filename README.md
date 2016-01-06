# docker-junit-rule

A junit rule to run docker containers

## Usage

Extend the `DockerContainerRule`. Example for rabbitMQ:

```java
public class RabbitContainerRule extends DockerContainerRule {
    public static final String RABBIT_CONTAINER_IMAGE_NAME = "rabbitmq:management";

    public RabbitContainerRule() {
        // List the ports to open on the container.
        // They will automatically be bound to random unused ports on your host
        super(RABBIT_CONTAINER_IMAGE_NAME, new String[]{"5672", "61613", "15672"});
    }

    @Override
    protected void before() throws Throwable {
        super.before();
        // wait for container to boot
        waitForPort(getRabbitServicePort());
    }

    public int getRabbitServicePort() {
        return getHostPort("5672/tcp");
    }

    public int getRabbitManagementPort() {
        return getHostPort("15672/tcp");
    }
}
```

Use it in your tests:

```java
import com.rabbitmq.client.ConnectionFactory;
import org.junit.ClassRule;
import org.junit.Test;
import rules.RabbitContainerRule;

public class RabbitIntegrationTest {

    @ClassRule
    public static RabbitContainerRule rabbitContainerRule = new RabbitContainerRule();

    @Test
    public void testConnectsToDocker() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(rabbitContainerRule.getDockerHost());
        factory.setPort(rabbitContainerRule.getRabbitServicePort());
        factory.newConnection();
    }
}
```

## Installation

The library is available on jcenter

### Maven

Add the following to your `pom.xml`:

```xml
<repositories>
    <repository>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <id>central</id>
        <name>bintray</name>
        <url>http://jcenter.bintray.com</url>
    </repository>
</repositories>

...

<dependency>
    <groupId>com.github.geowarin</groupId>
    <artifactId>docker-junit-rule</artifactId>
    <version>1.0.2</version>
</dependency>
```

### Gradle

Add the following to your `build.gradle`:

```groovy
repositories {
  mavenCentral()
}

dependencies {
  compile 'com.github.geowarin:docker-junit-rule:1.0.2'
}
```

## Principle

Uses https://github.com/spotify/docker-client to connect to the docker daemon API.

You should probably set the `DOCKER_HOST` and `DOCKER_CERT_PATH` on your machine.
If they are not set and your are not on UNIX, the client will try to connect to `https://192.168.99.100:2376`,
which is the adress of my default docker machine.
It works great for me but your mileage may vary.
