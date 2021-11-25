# Mass upload into artifactory

Working with maven needs access to an artifactory.  
In most cases, you will work with `mvn` which will download dependencies you specified in your `pom.xml` into your `.m2/repository`-directory.

But in some cases, you will need to upload your own artifacts. Maybe you want to share your code with the world or you simply want to migrate from another artifactory?

## Our example
For our example we will have
1. an artifactory located under `https://repo.my-company.com`
2. a repository `company-dependencies-maven-3d-party` in this artifactory
3. a username "Brad"
4. a password "kek"

## What we need

### An artifactory and credentials

Nothing to explain I hope.

### Curl

Install curl. Maybe [this article on stackoverflow](https://stackoverflow.com/questions/9507353/how-do-i-install-and-use-curl-on-windows) will help.

### Something to upload

We need something that is structured like an `.m2/repository`-directory. 
For example: Your actual `.m2/repository`-directory or the stuff you got by [downloading a whole artifactory](downloadWholeRepoFromArtifactory.md).

You can find your `.m2/repository`-directory by executing

```shell
cd "~/.m2/repository"
```
in any terminal.

E.g.: Brad surely would find his `.m2/repository`-directory on windows in `C:\Users\Brad\.m2\repository`.

Your `.m2`-directory should have a structure like this (containing the folder `repository`):
* .m2
  * repository
    * com
      * ...
    * org
      * company
        * company-module
          * 1.0.1
            * company-module-1.0.1.jar
            * company-module-1.0.1.pom
            * ...
          * ...
        * ...
      * ...
    * ...
  * ...

## Execute the mass upload

To execute the mass upload, you have to open a terminal and navigate to the directory where you want to upload from. 

As mentioned above: If you actually want to upload your `.m2/repository`-directory you can access it by executing

```shell
cd "~/.m2/repository"
```

Now copy this line:

```shell
find . -not -path "*/\.*" -type f -not -name maven-metadata\* -not -name archetype-catalog.xml\* -not -name \*\.lastUpdated -not -name _\*\.repositories -not -name '*.[a-z]*[[:digit:]]' | while read artifact; do curl -fsS -o /dev/null -u <USERNAME>:<PASSWORD> -H "X-Checksum-Sha1:$(sha1sum "${artifact}" | awk '{print $1}')" -X PUT "https://<ARTIFACTORY_URL>/artifactory/<REPOSITORY>/${artifact#*/}" --upload-file "${artifact}" || echo "${artifact}"; done
```

Then replace
- `<USERNAME>`
- `<PASSWORD>`
- `<ARTIFACTORY_URL>`
- `<REPOSITORY`

and **execute it in your terminal**.

Brad for example would execute
```shell
find . -not -path "*/\.*" -type f -not -name maven-metadata\* -not -name archetype-catalog.xml\* -not -name \*\.lastUpdated -not -name _\*\.repositories -not -name '*.[a-z]*[[:digit:]]' | while read artifact; do curl -fsS -o /dev/null -u Brad:ked -H "X-Checksum-Sha1:$(sha1sum "${artifact}" | awk '{print $1}')" -X PUT "https://repo.my-company.com/artifactory/company-dependencies-maven-3d-party/${artifact#*/}" --upload-file "${artifact}" || echo "${artifact}"; done
```

---
## Understanding the command line

### Find

Finding all relevant files is done by executing

```shell
find . -not -path "*/\.*" -type f -not -name maven-metadata\* -not -name archetype-catalog.xml\* -not -name \*\.lastUpdated -not -name _\*\.repositories -not -name '*.[a-z]*[[:digit:]]'
```

***You can actually execute this before the actual upload in advance to check which files are uploaded.*** 

The `-not -path` arguments will exclude the following: 
* `*/\.*` (current folder)
* `archetype-catalog\*`
* things that are created on upload
  * `maven-metadata\*` (created by the artifactory)
  * `'*.[a-z]*[[:digit:]]'` (all `.md5`, `sha1` und `sha256` - files)
* things that are created on download (via maven)
  * \*\.lastUpdated (time and details of the download)
  * _\*\.repositories (mainly  `_remote.repositories` - files, that are just containing information of the repository the dependency was downloaded from) 
beim Download von maven erstellt
  
### Simple Upload

A simple upload is done by 
```shell
curl -u username:password -T <PATH_TO_FILE> "https://<ARTIFACTORY_SERVER>/<REPOSITORY_PATH>/<TARGET_FILE>"
```

# What next?

Maybe you want to check how to [download a whole repository from an artifactory](downloadWholeRepoFromArtifactory.md)?