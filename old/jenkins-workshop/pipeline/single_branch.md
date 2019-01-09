
## 1. Making the pipeline script work

Now you have made a really nice pipeline in Jenkins just using the normal jobs.
Now we want it *as code*!

First off, we need a new `Pipeline` job.

* Click on `New Item`, choose `Pipeline` type, and give it a name.
* Head down to the `Pipeline` section of the job, and click on the "try sample pipeline" and choose `Hello world`
* Save and Build it.

The result should very well be that you have a blue (succesful) build, and in the main view a text saying the following will appear:

>This Pipeline has run successfully, but does not define any stages. Please use the stage step to define some stages in this Pipeline.

We have to look into that now, *don't we?*

## 2. Convert your pipeline

In pipeline, we like `stages` as they give us the ability to see where in the process things are going wrong.
So take a look at your old build script and transfer the things you did there to the jenkins script.

If you cant remember the syntax for creating stages, then here is the hello world example of it:

```groovy
node {
    stage ('Hello'){
        echo 'Hello World'
    }
}
```

Make three stages that does the following:

* `Preparation`: Clone the repository from git.
* `Build` : Executes maven `clean package`
* `Results` :  Make jUnit display the results of `**/target/surefire-reports/TEST-*.xml`, and archive the generated jar file in the `target` folder

Run this to see that it's working. The archiving part can be verified by looking for a small blue arrow next to the build number in the overview. Make sure you get your Jar file with you there.

## 2. Archiving

We also need to get the javadoc generated for the project.

Fortunately that can be done with a small `mvn site` command.

* Create another stage called `Javadoc` where you execute the above command, and archive the result in the `target/javadoc` folder.
* Archive the `target/gildedrose-*.jar` as well
