# Upload Artifacts build by jenkins (and clean old ones)

Sometimes you want to download the artifacts from jenkins. That's ok, because why build it on your own machine? 

But if you want to constantly upload, it surely is good to clean up old artifacts (e.g. after 30 days).
Also, you maybe don't want to upload every artifact, just the ones of branches named "release" or "master" or alternatively commits tagged with "upload" or "cleanOld".

## Before we start

You should jave
* some basic knowledge of a Jenkins-Pipeline
* (of course) a Jenkins-Server
* (of course) an Artifactory
* some user credentials (e.g. for a technical user) 
  * if not, see [here](https://docs.cloudbees.com/docs/cloudbees-ci/latest/cloud-secure-guide/injecting-secrets) how to inject secrets
  * also make sure the credentials will suffice your access your Artifactory
  
## Code
Copy the stages of the following code template and copy them into your Jenkins-File:

```groovy
pipeline {
    agent any
    tools {
        // here comes your tools
    }
    stages {
        // here comes your previous steps, ** you must build your artifact here**  
        stage('upload') {
            when {
                anyOf{
                    expression { branch_name =~ '.*release|.*master' }
                    tag 'upload'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '<CREDENTIAL_ID>', usernameVariable: 'artifactory_username', passwordVariable: 'artifactory_password')]) {
                        def (int code) = sh(
                                script: ''' 
                                        find . -not -path "*/\\.*" -type f -path \'*<PATH_TO_YOUR_TARGET_DIR>*\' -name \'<NAME_OF_YOUR_ARTIFACT>\' |
                                        while read artifact
                                        do 
                                            NAME="$(basename ${artifact%.*})-$(date +%Y-%m-%d_%H-%M-%S).$(basename ${artifact##*.})"
                                            curl -fsS -o /dev/null -w \'\\n%{response_code}\' -u $artifactory_username:$artifactory_password -H "X-Checksum-Sha1:$(sha1sum "${artifact}" | awk \'{print $1}\')" -X PUT "<ARTIFACTORY_SERVER>/artifactory/<REPOSITORY_PATH>/${NAME}" --upload-file "${artifact}" || echo "failed at ${artifact}"
                                        done
                                        ''' ,
                                returnStdout: true
                        ).trim().tokenize("\n")

                        echo "HTTP status code: $code"

                        if (code == 200 || code == 201) {
                            sh 'echo uploaded'
                        } else {
                            sh 'echo failed'
                            sh 'false'
                            error 'Pipeline stopped'
                        }
                    }
                }
            }
        }
        stage('delete old snapshots') {
            when {
                anyOf{
                    expression { branch_name =~ '.*release|.*master' }
                    tag 'cleanOld'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '<CREDENTIAL_ID>', usernameVariable: 'artifactory_username', passwordVariable: 'artifactory_password')]) {
                        echo "Deleting old Snapshots."
                        def results = sh(
                                script: '''
                                        END_TIME=$(($(date --date="30 days ago" +%s%N)/1000000)) 
                                        RESULTS=`curl -s -X GET -u $artifactory_username:$artifactory_password "https://<ARTIFACTORY_SERVER>/artifactory/api/search/creation?from=0&to=$END_TIME&repos=<REPOSITORY_NAME>"| grep uri | awk \'{print $3}\' | sed s\'/.$//\' | sed s\'/.$//\' | sed -r \'s/^.{1}//\'| sed  s\'/\\/api\\/storage//\'`
                                                                        
                                        for RESULT in $RESULTS
                                        do
                                            echo "deleting from $RESULT"
                                            curl -fsS -u $artifactory_username:$artifactory_password -X "DELETE" $RESULT
                                        done
                                        ''' ,
                                returnStdout: true
                        ).trim().tokenize("\n")

                        echo "Deleting results (please check success manually): $results"
                    }
                }
            }
        }
    }
}
```

Now replace
- `<CREDENTIAL_ID>` by your credentials (see also [here](https://docs.cloudbees.com/docs/cloudbees-ci/latest/cloud-secure-guide/injecting-secrets) how to inject secrets)
- `<PATH_TO_YOUR_TARGET_DIR>` to the artifact that was built in your previous steps, you may use wildcards (in Java it will be most commonly suffice to set `*/target*`) 
- `<NAME_OF_YOUR_ARTIFACT>` should be the name of your artifact which was built, you may use wildcards (in Java it will be most commonly suffice to set something like `my-project*-SNAPSHOT*`)
- `<ARTIFACTORY_SERVER>` by your artifactory server, e.g. `repo.my-company.com`
- `<REPOSITORY_PATH>` and `<REPOSITORY_NAME>` by your snapshot-repository where you want the upload to be placed (which are equal 90% of the time, e.g. `company-dependencies-maven-snapshots`)

You should be done. 

## What it does
As mentioned above: 
1. The first stage (uploading) only will apply 
   - for branches which names do end with "release" or "master",
   - or alternatively commits tagged with "upload" . 
2. The second stage (cleaning old artifacts) only will apply  
   - for branches which names do end with "release" or "master", 
   - or alternatively commits tagged with "cleanOld".

In much simpler words: 
- If you run your "master"- branch, your Jenkins will run both stages. That will suffice for 80% of your needs.
- For your "release"-branches, you just have to name them correctly to run both stages. That will suffice for another 15% of your needs.
- For any other case, just tag a commit with "upload" or "cleanOld" to run your job pipeline with one of the stages.

After running the jenkins pipeline with the first stage, there should be a new artifact containing a new artifact, which should look like the one you have whenever you build lokally (e.g.` my-project-1.0-SNAPSHOT.jar`), **but with the current date and time before the file extension** (e.g. `my-project-1.0-SNAPSHOT-2021-11-25_10-00-14.jar`).

If you run the jenkins pipeline 30 days later with the second stage, this very artifact should be deleted while newer ones do remain. 

## What if the upload does not work?
Check if you can upload at all by using

```shell
curl -u username:password -T <PATH_TO_FILE> "https://<ARTIFACTORY_SERVER>/artifactory/<REPOSITORY_PATH>/<TARGET_FILE>"
```

Maybe your credentials do not work on your artifact. 
Otherwise (= if this "manual" upload works), your credentials might not be injected properly. 
If that's not the case, you definitely should check the console output. 

### But if it uploads stuff I don't want?
If there are too many files, you may want to re-evaluate the line which says  `find . -not -path "*/\\.*" -type f -path \'*<PATH_TO_YOUR_TARGET_DIR>*\' -name \'<NAME_OF_YOUR_ARTIFACT>\'`.
You might want to add use something like `-not -name \\*.unwantedFileEnding` or similar.

## What if the deletion does not work
That's a tricky thing. 

Check out `https://<ARTIFACTORY_SERVER>/artifactory/api/search/creation?from=0&repos=<REPOSITORY_NAME>` (while replacing `<ARTIFACTORY_SERVER>` and `<REPOSITORY_NAME>`).

Now look at the `uri`s and compare them with the URIs which are linked in your artifactory (to those artifacts). 
They will most likely differ. You have to modify the `sed`-part (at the end of the line that begins with ``RESULTS=`curl -s -X GET``) to make them match. 
Since this is not trivial, I can't write down one solution for everybody. 