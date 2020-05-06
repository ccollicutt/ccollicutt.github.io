---
layout: post
title: GraalVM Native Images
categories:
header_image: "/img/graalvm.jpg"
header_permalink: "https://unsplash.com/photos/oOS_Lm4mXo0"
summary: "GraalVM Native Images allows the creation of Java programs contained in a single binary."

---

# {{ page.title }}

People are often concerned about the startup speed of Java applications. I don't think it matters for most use cases, but there are, of course, some situations where a very fast startup time and less memory usage would be useful...say small microservices. [GraalVM Native Images](https://www.graalvm.org/docs/reference-manual/native-image/) can, in certain situations, help with this.

>GraalVM Native Image allows you to ahead-of-time compile Java code to a standalone executable, called a native image. This executable includes the application classes, classes from its dependencies, runtime library classes from JDK and statically linked native code from JDK. It does not run on the Java VM, but includes necessary components like memory management and thread scheduling from a different virtual machine, called “Substrate VM”. Substrate VM is the name for the runtime components (like the deoptimizer, garbage collector, thread scheduling etc.). The resulting program has faster startup time and lower runtime memory overhead compared to a Java VM. - [GraalVM Docs](https://www.graalvm.org/docs/reference-manual/native-image/)

Dave Syer, a key, long time member of the Spring open source project, recently wrote a [post](https://spring.io/blog/2020/05/04/spring-cloud-function-native-images) discussing the performance of GraalVM Native Images and the JIT when using Spring Cloud Function with AWS Lambda.

>Using Spring Cloud Function is a very convenient way to develop functions that run on AWS and other platforms. If you also use the experimental Spring Graal Native Feature project to compile the result to a native binary executable they can run faster than the same application on a regular JVM. 

A couple reasons why you might use native images:

> Once compiled to a platform specific native-image applications should have very fast startup and a more reliable memory profile (no JIT causing memory spikes at the beginning). - [Spring Blog](https://spring.io/blog/2020/04/09/spring-graal-native-0-6-0-released)

## Compile using GraalVM Native Image

Let's build a native image with GraalVM.

I'll use sdkman to install the Java and GraalVM requirements.

```
sdk install java 20.0.0.r8-grl
```

Also we need gcc and zlib1g-dev, at least on Ubuntu 18.04 anyways.

>For compilation native-image depends on the local toolchain, so please make sure: glibc-devel, zlib-devel (header files for the C library and zlib) and gcc are available on your system. - [GraalVM docs](https://www.graalvm.org/docs/reference-manual/native-image/)

```
sudo apt install gcc zlib1g-dev -y
```

Clone the repo.

```
git clone https://github.com/spring-projects-experimental/spring-graal-native
cd spring-graal-native/spring-graal-native-samples/function-netty
```

Now I've got Java 8.

```
$ java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b06)
OpenJDK 64-Bit Server VM GraalVM CE 20.0.0 (build 25.242-b06-jvmci-20.0-b02, mixed mode)
```

And then use gu to install native-image.

```
$ gu install native-image
Downloading: Component catalog from www.graalvm.org
Processing Component: Native Image
Downloading: Component native-image: Native Image  from github.com
Installing new component: Native Image (org.graalvm.native-image, version 20.0.0)
```

At the root of the project, run `build.sh`.

```
$ pwd
/home/curtis/working/spring-graal-native
$ ./build.sh 
 ```

Then build the function-netty sample.

```
$ cd spring-graal-native-samples/function-netty
$ ./build.sh
```

I was initially building this on a VM with 8GB of memory and no swap, and it crashed out. I bumped the memory to 12GB and it built. I see a note in the native image docs about memory for builds, but couldn't quite grok it.

```
SNIP!
Error: Image build request failed with exit status 137
com.oracle.svm.driver.NativeImage$NativeImageError: Image build request failed with exit status 137
	at com.oracle.svm.driver.NativeImage.showError(NativeImage.java:1527)
	at com.oracle.svm.driver.NativeImage.build(NativeImage.java:1289)
	at com.oracle.svm.driver.NativeImage.performBuild(NativeImage.java:1250)
	at com.oracle.svm.driver.NativeImage.main(NativeImage.java:1209)

real	3m44.437s
user	11m4.659s
sys	0m17.460s
FAILURE: an error occurred when compiling the native-image.
```

With enough memory now, the build takes a few minutes:

```
$ ./build.sh 
=== Building function-netty sample ===
Packaging function-netty with Maven
Unpacking function-netty-0.0.1-SNAPSHOT.jar
Compiling function-netty with GraalVM Version 20.0.0 CE
SUCCESS
Testing executable 'function-netty'
SUCCESS
Build memory: 7.07GB
Image build time: 432.0s
RSS memory: 87.4M
Image size: 79.2M
Startup time: 0.178 (JVM running for 0.182)
```

Start up the function.

```
$ ./target/function-netty 

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                        

2020-05-05 14:12:00.458  INFO 9705 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication on tanzu-ubuntu-2 with PID 9705 (/home/curtis/working/spring-graal-native/spring-graal-native-samples/function-netty/target/function-netty started by curtis in /home/curtis/working/spring-graal-native/spring-graal-native-samples/function-netty)
2020-05-05 14:12:00.458  INFO 9705 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-05-05 14:12:00.515  INFO 9705 --- [           main] o.s.c.f.web.flux.FunctionHandlerMapping  : FunctionCatalog: org.springframework.cloud.function.context.catalog.BeanFactoryAwareFunctionRegistry@7f0b166628d8
2020-05-05 14:12:00.527  WARN 9705 --- [           main] io.netty.channel.DefaultChannelId        : Failed to find the current process ID from ''; using a random value: 398582704
2020-05-05 14:12:00.528  INFO 9705 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8080
2020-05-05 14:12:00.528  INFO 9705 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 0.081 seconds (JVM running for 0.083)
```

`0.081` seconds, that's pretty quick.


Curl it:

```
$ curl -s localhost:8080/ -d world -H "Content-Type: text/plain"; echo
hi world!
```

What is `target/function-netty`?

```
$ file target/function-netty
target/function-netty: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 3.2.0, BuildID[sha1]=680512d5d6ab33e499ecdb9a45d4b199841de0fb, with debug_info, not stripped
```

Size:

```
$ du -hsc target/function-netty
80M	target/function-netty
80M	total
```

Now I'll push it to Cloud Foundry (also now known as [Tanzu Application Service](https://tanzu.vmware.com/application-service) downstream-wise):

```
 $ cf push -b binary_buildpack -c target/function-netty cc-ni
Pushing app cc-ni to org Canada / space ccollicutt as ccollicutt@pivotal.io...
Getting app info...
Creating app with these attributes...
+ name:         cc-ni
  path:         /home/curtis/working/spring-graal-native/spring-graal-native-samples/function-netty
  buildpacks:
+   binary_buildpack
+ command:      target/function-netty
  routes:
+   cc-ni.cfapps.io

Creating app cc-ni...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 41.83 MiB / 41.83 MiB [===============================================================================================================================================================================] 100.00% 9s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloading binary_buildpack...
   Downloaded binary_buildpack
   Cell 850bb788-0cb8-4092-b839-32661cb636dc creating container for instance c3c3ad17-10a8-4885-a762-4b3ae5811cc4
   Cell 850bb788-0cb8-4092-b839-32661cb636dc successfully created container for instance c3c3ad17-10a8-4885-a762-4b3ae5811cc4
   Downloading app package...
   Downloaded app package (56.9M)
   -----> Binary Buildpack version 1.0.36
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading droplet...
   Uploading build artifacts cache...
   Uploaded build artifacts cache (214B)
   Uploaded droplet (56.4M)
   Uploading complete
   Cell 850bb788-0cb8-4092-b839-32661cb636dc stopping instance c3c3ad17-10a8-4885-a762-4b3ae5811cc4
   Cell 850bb788-0cb8-4092-b839-32661cb636dc destroying container for instance c3c3ad17-10a8-4885-a762-4b3ae5811cc4
   Cell 850bb788-0cb8-4092-b839-32661cb636dc successfully destroyed container for instance c3c3ad17-10a8-4885-a762-4b3ae5811cc4

Waiting for app to start...

name:              cc-ni
requested state:   started
routes:            cc-ni.cfapps.io
last uploaded:     Tue 05 May 15:15:33 EDT 2020
stack:             cflinuxfs3
buildpacks:        binary

type:            web
instances:       1/1
memory usage:    1024M
start command:   target/function-netty
     state     since                  cpu    memory    disk      details
#0   running   2020-05-05T19:15:47Z   0.0%   0 of 1G   0 of 1G   
```

And curl that:

```
$ curl -s cc-ni.cfapps.io -d "online curtis" -H "Content-Type: text/plain"; echo
hi online curtis!
```

Nice!

It's only using 40mb of memory.

```
$ cf app cc-ni
Showing health and status for app cc-ni in org Canada / space ccollicutt as ccollicutt@pivotal.io...

name:              cc-ni
requested state:   started
routes:            cc-ni.cfapps.io
last uploaded:     Tue 05 May 15:15:33 EDT 2020
stack:             cflinuxfs3
buildpacks:        binary

type:           web
instances:      1/1
memory usage:   1024M
     state     since                  cpu    memory        disk           details
#0   running   2020-05-05T19:15:48Z   0.5%   40.4M of 1G   114.7M of 1G   
```

## Push the Jar File

I can also just push the jar file instead of the native image binary.

```
$ cf push -p target/function-netty-0.0.1-SNAPSHOT.jar cf-ni-jar
```

Slightly more memory in use, about 110M more than the native image.

```
$ cf app cf-ni-jar
Showing health and status for app cf-ni-jar in org Canada / space ccollicutt as ccollicutt@pivotal.io...

name:              cf-ni-jar
requested state:   started
routes:            cf-ni-jar.cfapps.io
last uploaded:     Tue 05 May 15:30:15 EDT 2020
stack:             cflinuxfs3
buildpacks:        client-certificate-mapper=1.11.0_RELEASE container-security-provider=1.18.0_RELEASE
                   java-buildpack=v4.30-offline-https://github.com/cloudfoundry/java-buildpack.git#6986fd5
                   java-main java-opts java-security jvmkill-agent=1.16.0_RELEASE open-jdk...

type:           web
instances:      1/1
memory usage:   1024M
     state     since                  cpu    memory         disk           details
#0   running   2020-05-05T19:30:33Z   0.6%   158.3M of 1G   127.2M of 1G 
```

Also can curl it...

```
 $ curl -s cf-ni-jar.cfapps.io -d "online curtis" -H "Content-Type: text/plain"; echo
hi online curtis!
```

## Conclusion

So this is really interesting...the ability to create a single binary file for a Java and Spring app. I'd like to understand this better, especially what it is doing for 5 or 6 minutes while compiling, which also takes a considerable amount of RAM. I'm sure there is a lot of ongoing work, and that this will mostly be applicable in certain sitations, specifically cloud functions, but progress is definitely being made. Overall, faster startup time and lower memory usage will be extremely valuable.

There's a good article [here](https://dzone.com/articles/profiling-native-images-in-java) that discusses some of the pros and cons of native image.

It also appears that Java will take on building in some of this functionality itself in a project [called Leyden](https://mail.openjdk.java.net/pipermail/discuss/2020-April/005429.html).

>Leyden will address these pain points by introducing a concept of _static
images_ to the Java Platform, and to the JDK.
>* A static image is a standalone program, derived from an application,
    which runs that application -- and no other.
>* A static image is a closed world: It cannot load classes from outside
    the image, nor can it spin new bytecodes at run time.