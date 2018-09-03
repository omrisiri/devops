# Welcome

This workshop have three different parts:

* [The assignment](#coding-assignment) : if you do not know gilded rose, then read this before starting the programming exercise. If you know it, just skip this.
* [The setup](#setup) : Installs the tools on you Linux server that you need in order to run the assignment.
* [The exercises](#exercises) : The Jenkins excercises. When you have done the setup, then start here.

## Coding Assignment

While the purpose is to learn Jenkins, this will be a coding assignment. It is made just to give you some tangible code to work with:
Remember, this is not a programming exercise, but a Jenkins one; code is only there so you have something to build :)

This repository comes with a maven based java project from the start, but any language can be used. If you want to, just replace the java code with one of the other languages from (this repository)[https://github.com/emilybache/GildedRose-Refactoring-Kata].

## Gilded Rose Requirements Specification

Hi and welcome to team Gilded Rose. As you know, we are a small inn with a prime location in a
prominent city run by a friendly innkeeper named Allison. We also buy and sell only the finest goods.
Unfortunately, our goods are constantly degrading in quality as they approach their sell by date. We
have a system in place that updates our inventory for us. It was developed by a no-nonsense type named
Leeroy, who has moved on to new adventures. Your task is to add the new feature to our system so that
we can begin selling a new category of items. First an introduction to our system:

   - All items have a SellIn value which denotes the number of days we have to sell the item
   - All items have a Quality value which denotes how valuable the item is
   - At the end of each day our system lowers both values for every item

Pretty simple, right? Well this is where it gets interesting:

       - Once the sell by date has passed, Quality degrades twice as fast
       - The Quality of an item is never negative
       - "Aged Brie" actually increases in Quality the older it gets
       - The Quality of an item is never more than 50
       - "Sulfuras", being a legendary item, never has to be sold or decreases in Quality
       - "Backstage passes", like aged brie, increases in Quality as its SellIn value approaches;
	   Quality increases by 2 when there are 10 days or less and by 3 when there are 5 days or less but
   	   Quality drops to 0 after the concert

We have recently signed a supplier of conjured items. This requires an update to our system:

   - "Conjured" items degrade in Quality twice as fast as normal items

Feel free to make any changes to the UpdateQuality method and add any new code as long as everything
still works correctly. However, do not alter the Item class or Items property as those belong to the
goblin in the corner who will insta-rage and one-shot you as he doesn't believe in shared code
ownership (you can make the UpdateQuality method and Items property static if you like, we'll cover
for you).

Just for clarification, an item can never have its Quality increase above 50, however "Sulfuras" is a
legendary item and as such its Quality is 80 and it never alters.

## Setup

Head over to [setup readme](setup/README.md) to install jenkins on your server

## Exercises

### 0 Fork the repository

We need you to be able to push changes up to this repository as you go along. Therefore you need your own fork to play with.

* Fork of the [repository](https://github.com/praqma-training/jenkins-workshop) to obtain your own version of the code.

### 1 Authenticate Jenkins to GitHub

We want to make Jenkins talk to GitHub, in order for it to push and pull from your repositories. In order to do that, we need to set up an SSH key for the Jenkins server.

Tasks:

* Generate a new SSH key that will be used by Jenkins to prove itself to GitHub, by following the first part of [Generating a new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
* Add the public-key to your GitHub account by following [Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)
* Add the private-key to Jenkins, by opening your Jenkins server, and clicking `Credentials`
* Click `(global)` and `Add credentials`
* Choose Kind "SSH Username with private key", write the details used to generate the keypair and paste the contents from the private-key you generated in the first step, (default ~/.ssh/id_rsa)
      write the passphrase you chose.
* Save it.

### 2 make a freestyle job

Head over to the [freestyle readme](freestyle/README.md) and follow the instructions there to setup your first Jenkins job.

### 3 Implementing the Gilded Rose

Look in  `src/test/java/net/praqma/codeacademy/gildedrose/TexttestFixture.java` for examples of items to use for tests.

* Make a test and push it, observe it failing
* Make changes to pass the test and push them, observe as only working code are built to production

Do this a couple of times to get comfortable with how Jenkins schedules and works on commits.

### 4 Dockerize it

Now we have a fully functional pipeline, but it's not very nice to run `mvn` commands directly on the Jenkins machine. 

It requires each node that we are working on have maven installed, and we have nowhere documented what version of Java or Maven that is expected to compile the software.

So we want to run it in a controlled Docker container, making sure it's the same version everywhere and we won't need to worry about installing or managing Maven versions on our build nodes.

**Task:**

* Convert your `mvn` steps to run inside docker containers

* remove the build step `Invoke top-level Maven targets` that before took care of invoking Maven.
* Make a docker container run the build by adding a `Execute shell` build step with the following command: `docker run -i -u "$(id -u):$(id -g)" -v $PWD:/usr/src/mymaven -w /usr/src/mymaven --rm maven:3-jdk-8 mvn test`

> Info: if you are not familiar with all the ins and outs of docker, here is a small rundown of the command:
>* Use the `maven:3-jdk-8` docker image
>* Use the `-i` parameter to get std in and error out to the terminal. Stands for `interactive`.
>* `$PWD` will give you the path to the current directory. If you mount the current directory into the container and execute the command in that volume, it will be the same as running the command locally on the machine. The command is  `-v $PWD:/usr/src/mymaven -w /usr/src/mymaven`.
>* Add `--rm` to make the container delete itself when done executing. This is how you avoid old stopped containers filling up on the machine.
>*`mvn test` is the command to start maven and run the test goal.

* Click save, and then `build now`.
* Go into the console output like last time, and see that maven runs your tests, but in a docker container.
* Try to make a test fail, push the change, and see that the pipeline is still fully functional.

Now we are ready to take our code to next level infrastructure, making it *as code*, *versioned*, and finally able to maintain a green master with *pretested integration*!

### 5 code it

We want to reduce the amount of UI click to a bare minimum. Therefore we introduce a new job type `pipeline` to script the CI/CD workload. Head over to [pipeline/single_branch](pipeline/single_branch.md) to learn more.

### 6 version it

Can you remember how your pipeline looked like a year ago? Even if you do, having your pipeline versioned together with your code makes it a no-brainer building old revisions of your code. And it even works with multiple branches, so head over and make your pipeline [multibranch compliant](pipeline/multi_branch.md).

### 7 Pretested integration

Head over to [the pretested readme](pretested/README.md) to fulfill this part of the exercises.

**DONE!**

**That's it!** You rock at this!
