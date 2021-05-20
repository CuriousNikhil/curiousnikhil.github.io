---
layout: post
title: Publishing your Android/Kotlin library to maven-central in 2021
date: 2021-05-16 20:32:20 +0300
summary: Publish your android/kotlin library with default support provided by gradle with no fancy stuff.
img: https://cdn-images-1.medium.com/max/2000/0*CyYe_l54HdnZ6cph.png
---

There are lot of blogs available on how to publish the java/android library to maven-central but all of them are suggesting you to write pom files, include nexus-gradle plugin etc, and lot of complex stuff. But Gradle provides a way to publish your repository to maven-central out of the box. In this article, we are going to publish a library to maven central using Gradle’s provided plugins.

Artifact managers available today — Jitpack.io, Nexus Sonatype, Bintray (which is closed now but they have custom solutions available). We are going to use OSSRH Nexus Sonatype Repository Manager. This also makes the dependency integration easier to developers as gradle provides out of the box support for maven. Let’s start with preparing a release meta work.

### Step 1. Creating ticket on OSSRH

* Signup on [https://issues.sonatype.org/](https://issues.sonatype.org/)

* We will first create a jira ticket on OSSRH jira board to let sonatype know that we want to release a library

![](https://cdn-images-1.medium.com/max/3240/1*e3kO0kTvvhDkJunAIEAlDg.png)

* Write a summary

* Enter your group id — which can be your domain/company name

* Your project URL which can be github url. Then SCM url which again can be your .git URL

* If you want to add more than one contributor to your repository, you can add comma separated usernames (make sure they also have created account on OSSRH)

* Keep rest of the fields as default.

* After creating the ticket wait for some time to OSSRH comment on your ticket

![](https://cdn-images-1.medium.com/max/2776/1*gt-GWdd-pBkQzAsvV_cVaQ.png)

* They ask you to verify ownership of the domain(group-id) that you’ve entered.

* For this, either you can add a TXT record on your DNS or you can setup a redirect to your github page.

* In my case, my domain is on [Godaddy](https://www.godaddy.com/) so I created a TXT record in my DNS management portal.

![](https://cdn-images-1.medium.com/max/3436/1*zQKNPBiqt9_blmQ7OBSd7Q.png)

* Add comment on the ticket once done.

![](https://cdn-images-1.medium.com/max/2844/1*bILnHqS4-Sa5HXJXgwGNNg.png)

* OSSRH will add comment saying your space has been prepared you can now publish the snapshot/release artifacts on sonatype.

* Please note the approval may take a few minutes.

### Step 2. Generating gpg keys

Now we’ll setup gpg encryption.

* Create a key-pair by running following command gpg --gen-keys

* It’ll prompt you to select the key type —

    gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
      This is free software: you are free to change and redistribute it.
      There is NO WARRANTY, to the extent permitted by law.
      
      gpg: keyring `/N/u/username/Machine/.gnupg/secring.gpg' created
      gpg: keyring `/N/u/username/Machine/.gnupg/pubring.gpg' created
      Please select what kind of key you want:
         (1) RSA and RSA (default)
         (2) DSA and Elgamal
         (3) DSA (sign only)
         (4) RSA (sign only)
      Your selection?

Enter 1 to select the default key.

* GPG will prompt you to choose a keysize (in bits). Enter 2048.

* You will see: Enter 0for no expiration

    Requested keysize is 1024 bits
      Please specify how long the key should be valid.
               0 = key does not expire
            <n>  = key expires in n days
            <n>w = key expires in n weeks
            <n>m = key expires in n months
            <n>y = key expires in n years
      Key is valid for? (0)

* GPG will prompt for information it will use to construct a user ID to identify your key. It’s pretty straight forward just enter your name, email address, and a comment. Enter all information and then it’ll ask you for confirmation- hit O if you are okay with changes.

    You selected this USER-ID:
          "Full Name (comment) <username@<domain>.<something>>"
    
      Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?

* If you accept the user ID, GPG will prompt you to enter and **confirm a password**. Afterward, GPG will begin generating your key.

    gpg: key 09D2BD39 marked as ultimately trusted
      public and secret key created and signed.
      
      gpg: checking the trustdb
      gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
      gpg: depth: 0  valid:   4  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 4u
      gpg: next trustdb check due at <expiration_date>
      pub   1024R/09D2B839 2013-06-25 [expires: <expiration_date>]
            Key fingerprint = 6AB2 7763 0378 9F7E 6242  77D5 F158 CDE5 09D2 B839
      uid                  Full Name (comment) <username@something.edu>
      sub   1024R/7098E4C2 2013-06-25 [expires: <date>]

* To find your generated key hit gpg --list-keys

* The first line will be like pub XXXXX/YYYYYYYY <date>. The last 8 characterspart is your keyId.

* Now, publish your keys: (please change with your keyId). You can use any of the key-server but make sure you upload on hkp://keyserver.ubuntu.com

    $ gpg --keyserver hkp://keyserver.ubuntu.com --send-keys YYYYYYYY
    $ gpg --keyserver hkp://pgp.mit.edu --send-keys 09D2BD39

* To check if the keys are uploaded you can run

    $ gpg --keyserver hkp://pgp.mit.edu --search-keys <your-mail>

* Save your keys in secring.gpg file by running following — This file will be inside your ~/.gnupg folder.

    *gpg --export-secret-keys > ~/.gnupg/secring.gpg*

Meta work is done! Let’s write some code for publishing release.

### Step 3. Publish the library to Nexus artifact manager with Gradle publish plugin

We have generated few private keys and passwords which we don’t want to be get included with our Version Control. So we’ll use our machine’s ~/.gradle/ folder to store the credentials required to upload the artifact.

* Create a file ~/.gradle/gradle.properties. Since .gradle is shared with all the gradle projects, this gradle.properties file will be available to our library too.

* You’ve created an account on OSSRH jira board. We will need your username and password of OSSRH jira account.

* Add the following in your ~/.gradle/gradle.properties file and save the file.

    NEXUS_USERNAME=<OSSRH jira account username>
    NEXUS_PASSWORD=<OSSRH jira account password>

    signing.keyId=<your-key-id-(last 8 character of your key)>
    signing.password=<password-you-have-added-while-creating-gpg-key>
    signing.secretKeyRingFile=/Users/<username>/.gnupg/secring.gpg

Here NEXUS_USERNAME and NEXUS_PASSWORD are your jira account credentials these will be required while publishing the artifact to sonatype.

* Sonatype wants to sign the artifacts that you are publishing that’s why we have created the gpg keys.

* signing.keyId , signing.password , signging.secretKeyRingFile will be used by our Gradle’s signing plugin (which we will be using to sign our artifacts while publishing). Everything will make sense when we will write gradle maven-publish plugin configurations, hold on!!

Now, open your project and open your module’s (aka library)build.gradle file.

* Add the following plugins in the plugins{...}.Add your group-id and version name

    plugins 
        id 'java-library'
        id 'kotlin'

    //Add maven-publish and signing plugins
        id 'maven-publish'
        id 'signing'
    }
    //...
    group = '<groupId>'
    version = '<version>'

Here you can use the version and groupId variables which can be added to your app’s gradle.properties file or whichever way you want to keep them.

* Add a java task which will generate javadocs and sources jar

    plugins{
        id 'java-library'
        id 'kotlin'

    //Add maven-publish and signing plugins
        id 'maven-publish'
        id 'signing'
    }
    group = '<groupId>'
    version = '<version>'

    java {
        withJavadocJar()
        withSourcesJar()
    }

* Next, we need to configure the publishing — write a publishing gradle task just below the above java dsl block.

    publishing{
        publications{
            mavenJava(*MavenPublication*){

               artifactId = '<libraryname>'

                from components.java
    
                pom {
                    name = '<libraryname>'
                    description = '<description>'
                    url = '<library/project url>'
                    licenses {
                        license {
                            name = '<Your license>'
                            url = '<license-url>'
                        }
                    }
                    developers {
                        developer {
                            id = '<developerID>'
                            name = 'Developer Name'
                            email = 'Developer email'
                        }
                    }
                    scm {
                    connection = 'scm:git:git://github.com/CuriousNikhil/simplepoller.git'
                        developerConnection = 'scm:git:ssh://github.com/CuriousNikhil/simplepoller.git'
                        url = 'https://github.com/CuriousNikhil/simplepoller'
                    }
                }
            }
        }

        repositories {
            maven {
    
                credentials {
                    username = "$NEXUS_USERNAME"
                    password = "$NEXUS_PASSWORD"
                }
    
                name = "<name anything>"
                url = 'https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/'
            }
        }
    }

              
>  Here, I’m uploading the kotlin library (a jar). If you want to upload the aar android library you can tweak this publishing task. Check out the official doc from [https://developer.android.com/studio/build/maven-publish-plugin](https://developer.android.com/studio/build/maven-publish-plugin)

I’ve to write and configure my pom file to publish a jar. Gradle provides out of the box support for writing & configuring pom files for your jar. I’ve added my required configuration in pom{..} DSL. [These](https://docs.gradle.org/current/dsl/org.gradle.api.publish.maven.MavenPom.html) are the available options.

Next block you can see is of repositories. We have defined the credentials{...} with username and password and as we’ve set our required credentials in ~/.gradle/gradle.properties, those variables can be used here.

The URL for the sonatype I used is for the release artifact publishing. If you want to release SNAPSHOT you can use snapshot URL from sonatype. These URLs will be mentioned on the jira ticket when the space is created for you on sonatype.

For snapshot release you can configure this way —

    repositories {
            maven {
                def releasesRepoUrl = '<release-repo-url>'
                def snapshotsRepoUrl = '<snapshot-repo-url>'
           url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            }
        }

Publishing to local maven and more configurations are available in [**gradle-publishing plugin](https://docs.gradle.org/current/userguide/publishing_maven.html).**

* Now add the signing configurations in the signing{...} dsl. Write this just below our publishing dsl block.

    signing{
        sign publishing.publications.mavenJava
    }

As we have already added the required credentials in our ~/.gradle/gradle.properties those will be picked up while signing our publication. You can check more configurations available for [**gradle-signing plugin](https://docs.gradle.org/current/userguide/signing_plugin.html)**
>  You can check complete build.gradle file for reference [here](https://github.com/CuriousNikhil/simplepoller/blob/main/simplepoller/build.gradle).

And we are done!!! Now you can publish your repository. Just run —

    $ ./gradlew publish

* This will stage the artifact on the Nexus Sonatype.

* You need to login to OSSRH available at [**https://s01.oss.sonatype.org/](https://s01.oss.sonatype.org/)** in order to access and work with your staging repositories. Use the username and password from your account for the JIRA issue tracking system for OSSRH and the *Login* link in the top right hand corner of the OSSRH user interface.

* When you log in — jump to the staging repositories tab and you’ll see your artifact listed

![](https://cdn-images-1.medium.com/max/2000/0*vkYNA6JOIZLz6_0E.png)

* This is for you to verify if all the data is correct. The artifact, at this stage, is not released yet. You’ve to do that from here.

* If you are fine with the content and validated required data. You need to close this repository. Select the repository and hit close button. If there are no errors, your repository will be closed.

* Now you can hit on Release button to release and drop the artifact. If everything goes fine. Your library is released!! Yay!!

* (It’ll take some time to get on the CDN meanwhile you can add comment on ticket that you‘ve released your first component so OSSRH will start syncing to maven central)

* You can check gradle’s [customization option](https://docs.gradle.org/current/userguide/publishing_customization.html)

* If you want more automated release you can check Nexus Sonatype’s publishing gradle plugin [https://github.com/Codearte/gradle-nexus-staging-plugin](https://github.com/Codearte/gradle-nexus-staging-plugin)

Go publish your awesome libraries! ✌️

Special Thanks to — Gradle and Sonatype
