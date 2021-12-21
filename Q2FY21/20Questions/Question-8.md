# Q&A: The one where you're upgrading  spring-boot maven dependencies for enterprise edition

If you've been using the Spring-boot community edition, but you've decided that it's time to step up to the Enterprise Edition, then there a re a few things you'll need to do. We will step thtough them here.

## First things first, upgrade your database

One of the first differences between the CE and EE version is that there may have been some patch releases added. These sometimes require small changes to the database. You can always check [here](https://docs.camunda.org/manual/7.15/update/patch-level/) for the list of all patch releases. If this is the case you would just find and run the scripts against the database. The CE version can still run on the Database that has been updated for EE.

## Next up, upgrade your spring-boot maven dependencies

In your `pom.xml` file you will need to change a few dependencies.

```xml
<dependency>
  <groupId>org.camunda.bpm</groupId>
  <artifactId>camunda-bom</artifactId>
  <version>7.16.0-ee</version>
  <scope>import</scope>
  <type>pom</type>
</dependency>
```

Notice the `-ee` added to the end of the version tag. That's the important bit. You might also want to take this opportunity to upgrade whatever version you're using to the most recent, but that's up to you.

There are a couple of other dependencies you will also need to update:

```xml
<dependencies>
    <dependency>
      <groupId>org.camunda.bpm.springboot</groupId>
      <artifactId>camunda-bpm-spring-boot-starter-rest</artifactId>
      <version>7.16.0-ee</version>
    </dependency>

    <dependency>
      <groupId>org.camunda.bpm.springboot</groupId>
      <artifactId>camunda-bpm-spring-boot-starter-webapp-ee</artifactId>
      <version>7.16.0-ee</version>
    </dependency>
  ...
</dependencies>
```

And then finally, and crucially, you'll need to make sure that your `repositories` section is up to date:

```xml
<repositories>
  <repository>
    <id>camunda-bpm-nexus-ee</id>
    <name>camunda-bpm-nexus</name>
    <url>
      https://camunda.jfrog.io/artifactory/private/
    </url>
  </repository>
</repositories>
```

That will get all your dependndecies up to date, but with your enterprise license, you should also have gotten a username and password to use to download the Enterprise Edition. YOu'll need that in order for your `.pom.xml. to access the repositories you just defined.

In order to do that, check in your `.m2` directory for a `settings.xml` file. If it's not there, you can create it with the following structure:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>
    <server>
      <id>camunda-bpm-nexus-ee</id>
      <username>YOUR_USERNAME</username>
      <password>YOUR_PASSWORD</password>
    </server>
  </servers>

</settings>
```

Now when you run `mvn clean install` your project should pull the dependencies from the EE version and build your application properly.

