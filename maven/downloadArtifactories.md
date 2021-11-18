# How to download an artifactory

Working with maven needs access to an artifactory.  
In most cases, you will work with `mvn` which will download dependencies you specified in your `pom.xml` into your `.m2`-directory.
But sometimes you want to download a whole artifactory. Maybe you want to migrate from an old one or don't know how to mirror?

Btw: Thanks to Peter Sheng for [this article](https://shenxianpeng.github.io/2021/05/artifactory-api-search/). I basically copied its code and made some smaller, but essential changes for my needs. 

## Our example
For our example we will have
1. an artifactory located under `https://repo.my-company.com/` 
2. a repository `company-dependencies-maven` in this artifactory
3. a username "Brad"
4. a password "kek"

## Simple edition
In most cases it will suffice if you just call (via browser or as get Command) 
```shell
https://<ARTIFACTORY_URL>/artifactory/api/archive/download/<REPO_NAME>/?archiveType=zip
```

so in our case

```shell
https://repo.my-company.com/artifactory/api/archive/download/company-dependencies-maven/?archiveType=zip
```

That will download a zip. Just unzip it!

Sadly, in 9 out of 10 cases it will get you this:
```json
{
  "errors" : [ {
    "status" : 403,
    "message" : "Download Folder functionality is disabled."
  } ]
}
```

Well, go complicated then. 

## Complicated version

### What we need
1. The following code works on shellscript, so you might want to look to [Enable WSL2](/windows/windowsSubsystem2ForLinux.md) if you use windows. (Alternatively, you might want to use GitBash.)
2. You need to have installed curl.

### Step1: Create a script

Create a new file "download.sh" on your machine. Now copy the following content:

```shell
# download.sh
# call e.g. with .\download.sh LOGIN_NAME LOGIN_PASSWORD ARTIFACTORY_URL REPO_NAME
# you need to have access to the artifactory, so maybe try to login first (calling the ARTIFACTORY_URL via browser and try to log in) 
# everything from the repo will be downloaded into a new subfolder "downloaded" (subfolder of wherever you execute the script from)
USERNAME=$1
PASSWORD=$2
ARTIFACTORY=$3
REPO=$4

if [ ! -x "`which sha1sum`" ]; then echo "You need to have the 'sha1sum' command in your path."; exit 1; fi


RESULTS=`curl -s -X GET -u $USERNAME:$PASSWORD "$ARTIFACTORY/api/search/creation?from=0&repos=$REPO" | grep uri | awk '{print $3}' | sed s'/.$//' | sed s'/.$//' | sed -r 's/^.{1}//'`
echo $RESULTS

for RESULT in $RESULTS ; do
    echo "fetching path from $RESULT"
    PATH_TO_FILE=`curl -s -X GET -u $USERNAME:$PASSWORD $RESULT | grep downloadUri | awk '{print $3}' | sed s'/.$//' | sed s'/.$//' | sed -r 's/^.{1}//'`
    OUTPUT_PATH="./downloaded${PATH_TO_FILE#*$REPO}" # gets everything after REPO and adds "./downloaded" as prefix, so e.g. http://my-artifactory/my-repo/path/to/file.jar  becomes ./downloaded/path/to/file.jar

	echo "download file to $OUTPUT_PATH"
  curl -u $USERNAME:$PASSWORD --create-dirs -o $OUTPUT_PATH -O $PATH_TO_FILE
  echo "#####"
done
```

### Step 2: Execute the script
In your terminal, navigate into the folder where your script is located. 

Execute 
```shell
.\download.sh <LOGIN_NAME> <LOGIN_PASSWORD> <ARTIFACTORY_URL> <REPOSITORY>
```

So in our example
```shell
 .\download.sh Brad kek https://repo.my-company.com company-dependencies-maven                                 
```

### Step 3: Wait
This will take a long, long, long, long time - depending on the size of the repository. 

### Step 4: Check the output
Everything just downloaded should be set into a folder "downloaded" (as a subfolder from where you executed the skript).
