# Freestyle jobs

Freestyle jobs were for a long time the way you would make Jenkins build your code, and it still has its merits.

It is rather easy to create a job and make your first build using these steps:

## 1. Create a job and clone git:

* Go into your Jenkins server and click on the `New Item` button on the left.
* Name your new job "gilded rose" and choose `Freestyle project` and click OK
* Under `Source Code Management` choose git, and paste in your git clone URL for this project (Remember to use the _ssh_-url to _your repository_!).
* Choose the credentials that you have set up in Jenkins to auth it against GitHub.
* Click `Save` and then the `Build Now` button.
* Observe that there is a new build in the build history, that hopefully is blue.
* Clik on it and click on `Console Output` to see something like this on your screen :

```
Started by user admin
Building in workspace /var/jenkins_home/workspace/my first job
 > git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url git@github.com:figaw/gildedrose.git # timeout=10
Fetching upstream changes from git@github.com:figaw/gildedrose.git
 > git --version # timeout=10
using GIT_SSH to set credentials Test to ssh jenkins access github
 > git fetch --tags --progress git@github.com:figaw/gildedrose.git +refs/heads/*:refs/remotes/origin/*
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision 06344e7eb74250449756084692ce55c4e701ce7d (refs/remotes/origin/master)
Commit message: "Update README.md"
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 06344e7eb74250449756084692ce55c4e701ce7d
First time build. Skipping changelog.
Finished: SUCCESS
```

**Congratulations, you have now made your first jenkins job!**

Well... You need to actually build the software, but at least you have it cloned now.

## 2. Running a maven test

* Click on the `Back to Project` button, and go in and `Configure` the job again.
* * Under the `Build` section, add an `Invoke top-level Maven targets` step and write `test` in it. That will trigger the maven test goal on the project, compiling the java code and running the unit tests.
* Click save, and build now once more.
* The build should go red, indicating an error.
* Go into the console output like last time, and see that maven now actually runs your tests.

The output should be something like this:

```bash
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running net.praqma.codeacademy.gildedrose.GildedRoseTest
Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 0.046 sec <<< FAILURE! - in net.praqma.codeacademy.gildedrose.GildedRoseTest
foo(net.praqma.codeacademy.gildedrose.GildedRoseTest)  Time elapsed: 0.009 sec  <<< FAILURE!
org.junit.ComparisonFailure: expected:<[bar]> but was:<[foo]>
	at org.junit.Assert.assertEquals(Assert.java:115)
	at org.junit.Assert.assertEquals(Assert.java:144)
	at net.praqma.codeacademy.gildedrose.GildedRoseTest.foo(GildedRoseTest.java:14)


Results :

Failed tests: 
  GildedRoseTest.foo:14 expected:<[bar]> but was:<[foo]>

Tests run: 1, Failures: 1, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.263 s
[INFO] Finished at: 2018-06-28T13:13:39+02:00
[INFO] Final Memory: 16M/191M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.17:test (default-test) on project gildedrose: There are test failures.
[ERROR] 
[ERROR] Please refer to /var/lib/jenkins/slave/workspace/freestyle/target/surefire-reports for the individual test results.
[ERROR] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
Build step 'Invoke top-level Maven targets' marked build as failure
Finished: FAILURE
```

This is OK for now, we will fix the code in a short while.

### 3. Scheduling the build

As a team, you do not want to go in and manually build the project every time you have some new code committed. _it needs to be automated, right!?!_
So we want Jenkins to pull for changes, and build whenever something new is happening.

* Go into `Configuration` again and select the `Poll SCM` checkbox
* Type in `* * * * */1` to tell Jenkins to check for new commits every minute.
* Make a new commit, commenting the test in src/test/java/net/praqma/codeacademy/gildedrose/GildedRoseTest.java
* Push that change to GitHub, and monitor as Jenkins starts a build automatically.
* Note that the build still fails (because the test is failing) _this is OK_.

### 4. Generating an artifact

Our Java project needs to be packaged into a Jar file, in order to be ready for release.

* Change the maven goal from `test` to `install`.
* Under `Post-build Actions` add the `Archive the artifacts` action, and write `target/gildedrose-*.jar` in it. That will take the output from the maven goal and add it as an artifact.
* Choose the advanced options and select `Archive artifacts only if build is successful` as well to reduce the number of artifacts
* Click save
* Go back to the job dashboard.
* Fix the unit test by implementing a dumb way of solving the test.
* Push the change to GitHub, and monitor that Jenkins will grab that change and make a build, producing an artifact.

### 5. fixing Gilded Rose

Having your pipeline set up, now it is time to fix the software problem itself. Go back to [the main document and go through that](../README.md)