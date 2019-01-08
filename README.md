# Spring Boot Vue.js Example using Gradle
This repository contains a basic example project to show how to deliver a Vue.js application via a Spring Boot application.
Because most sample projects are based on Maven, I have created an example using Gradle. This example also works
for other Javascript frameworks.
This is a very basic executable version and in the following I have explained which steps have to be 
done to create an equivalent project.

# Installation steps from scratch
* Setup gradle project
* Use Spring Boot Initializr to set up the backend module
* Create the frontend module containing a gradle config

## Optional install NodeJs manual for easier development and initialization
* Install NodeJs and npm to setup vue-module using the vue-cli 3/ use it for development
* Install vue-cli using *npm install -g @vue/cli*
* Use *vue create frontend* to initialize the vue project
* Run *npm run serve* to start the local NodeJs server
 
# Configuration

## Adding node plugin
So that we can build without manually installing NodeJs, we use the plugin **com.moowork.node**.  
The plugin must be added to the Gradle config of the frontend module and configured properly.
It is important to set **nodeModulesDir = file("${project.projectDir}")** in the config of the plugin otherwise the 
package.json won't be found.  
The build of the Vue application is done using the command *npm run build* and creates a folder called **dist** containing 
the output resources of the Vue application. As default this folder will be created in the root of the module.  
Since in the Gradle world the output of a build is always contained in the **build** folder, we should also move the 
**dist** folder to */build*.

 To do this add the vue.config.js with the following config to the module root.
```javascript
 module.exports = {
   outputDir: 'build/dist'
 }
```

## Defining the landing page
In this basic example we additionally set the output dir to **'build/dist/public'**, so in the artifact it is included in *WEB-INF/classes/public*.  
Now **WelcomePageHandlerMapping** of spring-boot-autoconfigure will automatically set the index.html as welcome page.  
Of course the index.html can also be made available via a controller.

```javascript
 module.exports = {
   outputDir: 'build/dist/public'
 }
```

You can see this by the following line when starting the application.
```text
o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [public/index.html]
```

## Weaving frontend and backend build together
  
Create a Gradle task in the **frontend** module config that will delete the *build/dist* to get rid of old files and executes **npm run build** to build the Vue application. 
For the Vue application to be available as resources in the backend module, a SourceSet must be added in the Gradle config. 
This configures the dist folder in the output folder of the frontend build as srcDir.

```text
sourceSets {
    main {
        resources {
            srcDir '../frontend/build/dist'
        }
    }
}
```
 
It is necessary that the build of the backend depends on the frontend, because they are connected by the SourceSet. Therefore
in the build cycle of the backend the build task of the frontend depends on the task **processResources**.
```text
processResources.dependsOn ':frontend:npmBuild'
```

# Running the application
You can deploy your application on a server or start it using the **bootRun** task. When opening http://localhost:8080 the Vue application will be displayed. 

# Next Steps
You can now develop a REST API in the backend e.g. using [Spring Data REST](https://spring.io/projects/spring-data-rest) or [Spring MVC](https://spring.io/guides/tutorials/rest/).
Then you can use [axios](https://github.com/axios/axios) in the frontend to call the REST API.  
Of course you shouldn't forget about the [security](https://spring.io/projects/spring-security).

# Using proxy in development
If you start the Vue application in NodeJs during local development and want to access the REST API on another server 
(e.g. backend deployed on Tomcat), you can set up a proxy in the vue configuration. The documentation for configuring the 
proxy can be found [here](https://cli.vuejs.org/config/#devserver-proxy).
