For Manual Deployment
Pros
Cons
Dependencies are out of the final jar.
Copy Dependencies to a specific directory
````

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <executions>
    <execution>
      <id>copy-dependencies</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>copy-dependencies</goal>
      </goals>
      <configuration>
        <outputDirectory>${project.build.directory}/${project.build.finalName}.lib</outputDirectory>
      </configuration>
    </execution>
  </executions>
</plugin> 
````

Make the Jar Executable and Classpath Aware
```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <addClasspath>true</addClasspath>
        <classpathPrefix>${project.build.finalName}.lib/</classpathPrefix>
        <mainClass>${fully.qualified.main.class}</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
```
At this point the jar is actually executable with external classpath elements.
````
$ java -jar target/${project.build.finalName}.jar
````

### Make Deployable Archives
The jar file is only executable with the sibling ...lib/ directory. We need to make archives to deploy with the directory and its content.

````

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-antrun-plugin</artifactId>
  <executions>
    <execution>
      <id>antrun-archive</id>
      <phase>package</phase>
      <goals>
        <goal>run</goal>
      </goals>
      <configuration>
        <target>
          <property name="final.name" value="${project.build.directory}/${project.build.finalName}"/>
          <property name="archive.includes" value="${project.build.finalName}.${project.packaging} ${project.build.finalName}.lib/*"/>
          <property name="tar.destfile" value="${final.name}.tar"/>
          <zip basedir="${project.build.directory}" destfile="${final.name}.zip" includes="${archive.includes}" />
          <tar basedir="${project.build.directory}" destfile="${tar.destfile}" includes="${archive.includes}" />
          <gzip src="${tar.destfile}" destfile="${tar.destfile}.gz" />
          <bzip2 src="${tar.destfile}" destfile="${tar.destfile}.bz2" />
        </target>
      </configuration>
    </execution>
  </executions>
</plugin>
````
Now you have 
````

target/${project.build.finalName}.(zip|tar|tar.bz2|tar.gz) which each contains the jar and lib/*.
````

### Apache Maven Assembly Plugin
- Pros
- Cons

    No class relocation support (use maven-shade-plugin if class relocation is needed).
````

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-assembly-plugin</artifactId>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>single</goal>
      </goals>
      <configuration>
        <archive>
          <manifest>
            <mainClass>${fully.qualified.main.class}</mainClass>
          </manifest>
        </archive>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
      </configuration>
    </execution>
  </executions>
</plugin>
````

You have 
````
target/${project.bulid.finalName}-jar-with-dependencies.jar.
````
### Apache Maven Shade Plugin
- Pros
- Cons
````
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <shadedArtifactAttached>true</shadedArtifactAttached>
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>${fully.qualified.main.class}</mainClass>
          </transformer>
        </transformers>
      </configuration>
    </execution>
  </executions>
</plugin>
````
You have 
````
target/${project.build.finalName}-shaded.jar.
````
### onejar-maven-plugin
- Pros
- Cons

     Not actively supported since 2012.
````     
<plugin>
  <!--groupId>org.dstovall</groupId--> <!-- not available on the central -->
  <groupId>com.jolira</groupId>
  <artifactId>onejar-maven-plugin</artifactId>
  <executions>
    <execution>
      <configuration>
        <mainClass>${fully.qualified.main.class}</mainClass>
        <attachToBuild>true</attachToBuild>
        <!-- https://code.google.com/p/onejar-maven-plugin/issues/detail?id=8 -->
        <!--classifier>onejar</classifier-->
        <filename>${project.build.finalName}-onejar.${project.packaging}</filename>
      </configuration>
      <goals>
        <goal>one-jar</goal>
      </goals>
    </execution>
  </executions>
</plugin>
````
### Spring Boot Maven Plugin
- Pros
- Cons
       Add potential unecessary Spring and Spring Boot related classes.
       
````       
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>repackage</goal>
      </goals>
      <configuration>
        <classifier>spring-boot</classifier>
        <mainClass>${fully.qualified.main.class}</mainClass>
      </configuration>
    </execution>
  </executions>
</plugin>
````

You have 
````
target/${project.bulid.finalName}-spring-boot.jar.
````