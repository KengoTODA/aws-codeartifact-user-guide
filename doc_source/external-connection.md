# Add an external connection<a name="external-connection"></a>

You can add a connection between a CodeArtifact repository and an external, public repository such as [https://npmjs\.com](https://npmjs.com) or the [Maven Central repository](https://repo.maven.apache.org/maven2/)\. Then, when you request a package from the CodeArtifact repository that's not already present in the repository, the package can be fetched from the external connection\. This makes it possible to consume open\-source dependencies used by your application\.

**Topics**
+ [Add an external connection to a repository](#adding-an-external-connection)
+ [Supported external connection repositories](#supported-public-repositories)
+ [Remove an external connection](#removing-an-external-connection)
+ [Fetch npm packages from an external connection](#fetching-packages-from-a-public-repository)
+ [Fetch Maven packages from an external connection](#fetch-maven-packages-from-public-repo)
+ [npm ingestion behavior](#npm-ingestion-behavior)
+ [Maven ingestion behavior](#maven-ingestion-behavior)
+ [CodeArtifact behavior when an external repository is not available](#external-connection-unavailable)

## Add an external connection to a repository<a name="adding-an-external-connection"></a>

To add an external connection to a CodeArtifact repository, use `associate-external-connection`\.

```
aws codeartifact associate-external-connection --external-connection public:npmjs \
    --domain my_domain --domain-owner 111122223333 --repository my_repo
```

Example output:

```
{
    "repository": {
        "name": my_repo
        "administratorAccount": "123456789012",
        "domainName": "my_domain",
        "domainOwner": "111122223333",
        "arn": "arn:aws:codeartifact:us-west-2:111122223333:repository/my_domain/my_repo",
        "description": "A description of my_repo",
        "upstreams": [],
        "externalConnections": [
            {
                "externalConnectionName": "public:npmjs",
                "packageFormat": "npm",
                "status": "AVAILABLE"
            }
        ]
    }
}
```

**Note**  
A repository is limited to a single external connection only\.

## Supported external connection repositories<a name="supported-public-repositories"></a>

 CodeArtifact supports an external connection to the following public repositories\. To use the CodeArtifact CLI to specify an external connection, use the value in the **Name** column for the `--external-connection` parameter when you run the `associate-external-connection` command\. 


| Repository type | Description | Name | 
| --- | --- | --- | 
| npm | npm public registry | public:npmjs | 
| Python | Python Package Index | public:pypi | 
| Maven | Maven Central | public:maven\-central | 
| Maven | Google Android repository | public:maven\-googleandroid | 
| Maven | Gradle plugins repository | public:maven\-gradleplugins | 
| Maven | CommonsWare Android repository | public:maven\-commonsware | 
| NuGet | NuGet Gallery | public:nuget\-org | 

## Remove an external connection<a name="removing-an-external-connection"></a>

To remove an external connection, use `disassociate-external-connection`\.

```
aws codeartifact disassociate-external-connection --external-connection public:npmjs \
    --domain my_domain --domain-owner 111122223333 --repository my_repo
```

Example output:

```
{
    "repository": {
        "name": my_repo
        "administratorAccount": "123456789012",
        "domainName": "my_domain",
        "domainOwner": "111122223333",
        "arn": "arn:aws:codeartifact:us-west-2:111122223333:repository/my_domain/my_repo",
        "description": "A description of my_repo",
        "upstreams": [],
        "externalConnections": []
    }
}
```

## Fetch npm packages from an external connection<a name="fetching-packages-from-a-public-repository"></a>

After you add an external connection, configure your package manager to use your CodeArtifact repository\. Use the following for **`npm`**\.

```
aws codeartifact login --tool npm --domain my_domain --domain-owner 111122223333 --repository my_repo
```

Then, request the package from the public repository\.

```
npm install lodash
```

After the package has been copied into your CodeArtifact repository, you can use the `list-packages` and `list-package-versions` commands to view it\.

```
aws codeartifact list-packages --domain my_domain --domain-owner 111122223333 --repository my_repo
```

Example output:

```
{
    "packages": [
        {
            "format": "npm",
            "package": "lodash"
        }
    ]
}
```

The `list-package-versions` command lists all versions of the package copied into your CodeArtifact repository\. In some cases, this is all of the versions of the package in the external repository\. In other cases, this is a subset of those versions\. For more information, see [npm ingestion behavior](#npm-ingestion-behavior)\.

```
aws codeartifact list-package-versions --domain my_domain --domain-owner 111122223333 --repository my_repo --format npm --package lodash
```

Example output:

```
{
    "defaultDisplayVersion: "1.2.5"
    "format": "npm",
    "package": "lodash",
    "namespace": null,
    "versions": [
        {
            "version": "0.6.0", 
            "revision": "REVISION-1-SAMPLE-6C81EFF7DA55CC",
            "status": "Published"
        },
        {
            "version": "0.4.2",
            "revision": "REVISION-2-SAMPLE-6C81EFF7DA55CC",
            "status": "Published"
        },
        {
            "version": "0.6.1",
            "revision": "REVISION-3-SAMPLE-6C81EFF7DA55CC",
            "status": "Published"
        },
        {
            "version": "0.4.0",
            "revision": "REVISION-4-SAMPLE-6C81EFF7DA55CC",
            "status": "Published"
        }
    ]
}
```

## Fetch Maven packages from an external connection<a name="fetch-maven-packages-from-public-repo"></a>

 After you add an external connection, configure your build tool to use your CodeArtifact repository\. For more information, see [Use CodeArtifact with mvn](maven-mvn.md) and [Use CodeArtifact with Gradle](maven-gradle.md)\. If you run either tool \(for example, `gradle build`\), packages are requested from Maven Central and stored in your CodeArtifact repository\. 

### Restrict Maven dependency downloads to a CodeArtifact repository<a name="restrict-maven-downloads"></a>

 If a package cannot be fetched from a configured repository, by default, the `mvn` command fetches it from Maven Central\. Add the `mirrors` element to `settings.xml` to make `mvn` always use your CodeArtifact repository\.

```
<settings>
  ...
    <mirrors>
      <mirror>
        <id>central-mirror</id>
        <name>CodeArtifact Maven Central mirror</name>
        <url>https://my_domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/maven/my_repo/</url>
        <mirrorOf>central</mirrorOf>
      </mirror>
    </mirrors>
  ...
</settings>
```

If you add a `mirrors` element, you must also have a `pluginRepository` element in your `settings.xml` or `pom.xml`\. The following example fetches application dependencies and Maven plugins from a CodeArtifact repository\. 

```
<settings>
...
  <profiles>
    <profile>
      <pluginRepositories>
        <pluginRepository>
          <id>codeartifact</id>
          <name>CodeArtifact Plugins</name>
          <url>https://my_domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/maven/my_repo/</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
...
</settings>
```

The following example fetches application dependencies from a CodeArtifact repository and fetches Maven plugins from Maven Central\.

```
<profiles>
   <profile>
     <id>default</id>
     ...
     <pluginRepositories>
       <pluginRepository>
         <id>central-plugins</id>
         <name>Central Plugins</name>
         <url>https://repo.maven.apache.org/maven2/</url>
         <releases>
             <enabled>true</enabled>
         </releases>
         <snapshots>
             <enabled>true</enabled>
         </snapshots>
       </pluginRepository>
     </pluginRepositories>
   ....
   </profile>
 </profiles>
```

## npm ingestion behavior<a name="npm-ingestion-behavior"></a>

When a package is requested from a repository with an external connection to [https://npmjs\.com](https://npmjs.com), CodeArtifact ingests that package version and up to two versions of its direct and transitive dependencies\. This reduces the time to ingest the dependency tree\. 

 Each dependency has a specified version constraint\. For example, the npm package version `webpack 4.41.2` specifies a dependency on `@babel/core` with a version constraint of `^7.7.2`\. The caret \(^\) specifies that the latest minor or patch version is used \(for example, `7.7.4` or `7.8.0`\)\. When `webpack 4.41.2` is ingested, the most recent published version of `@babel/core` that satisfies the version constraint is ingested\. If the version of `@babel/core` specified by the `latest` tag is different, it is also ingested\. This logic applies to the direct and transitive dependencies of `@babel/core` and the direct and transitive dependencies of `webpack 4.41.2`\. 

If ingestion of a package is not complete in 40 seconds, a 404 error is returned to the client\.

```
npm ERR! code E404
npm ERR! 404 Not Found - GET https://my_domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/npm/my_repo/lodash - Ingestion is in progress. Please try again later.
npm ERR! 404
npm ERR! 404  'lodash@^4.17.15' is not in the npm registry.
npm ERR! 404 You should bug the author to publish it (or use the name yourself!)
npm ERR! 404
npm ERR! 404 Note that you can also install from a
npm ERR! 404 tarball, folder, http url, or git url.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/username/.npm/_logs/2019-09-22T01_47_43_155Z-debug.log
```

When this occurs, CodeArtifact is still copying packages from the external repository asynchronously\. Retry the same command to complete the ingestion of the entire dependency tree\.

## Maven ingestion behavior<a name="maven-ingestion-behavior"></a>

 When a Maven package is requested from a repository with an external connection to [Maven Central repository](https://repo.maven.apache.org/maven2/), CodeArtifact ingests all assets of the package version that follow the standard Maven asset naming conventions\. The dependencies of a package version are not ingested until they are requested by the client \(for example, `mvn`\)\. 

 If ingestion of a required asset is not complete within 40 seconds, a 404 error is returned to the client\. A timeout error during a Gradle build might look like the following\.

```
> Could not resolve all files for configuration ':compileClasspath'.
  > Could not resolve org.mockito:mockito-core:3.1.0.
    Required by:
        project :
      > Could not resolve org.mockito:mockito-core:3.1.0.
         > Could not get resource 'https://my_domain.codeartifact.aws.a2z.com/maven/my_domain/org/mockito/mockito-core/3.1.0/mockito-core-3.1.0.pom'.
            > Could not GET 'https://my_domain.codeartifact.aws.a2z.com/maven/my_domain/org/mockito/mockito-core/3.1.0/mockito-core-3.1.0.pom'.
```

 When this occurs, CodeArtifact is still copying packages from the external repository asynchronously\. Retry the same command to complete the ingestion of the entire dependency tree\. 

## CodeArtifact behavior when an external repository is not available<a name="external-connection-unavailable"></a>

Occasionally, an external repository will experience an outage that means CodeArtifact cannot fetch packages from it, or fetching packages is much slower than normal\. When this occurs, package versions already pulled from an external repository \(e\.g\. **npmjs\.com**\) and stored in a CodeArtifact repository will continue to be available for download from CodeArtifact\. However, packages that are not already stored in CodeArtifact may not be available, even when an external connection to that repository has been configured\. For example, your CodeArtifact repository might contain the npm package version `lodash 4.17.19 ` because that's what you have been using in your application so far\. When you want to upgrade to `4.17.20`, normally CodeArtifact will fetch that new version from **npmjs\.com** and store it in your CodeArtifact repository\. However, if **npmjs\.com** is experiencing an outage this new version will not be available\. The only workaround is to try again later once **npmjs\.com** has recovered\.

External repository outages can also affect publishing new package versions to CodeArtifact\. In a repository with an external connection configured, CodeArtifact will not permit publishing a package version that is already present in the external repository, see [Packages overview](packages-overview.md) for more information\. However, an external repository outage may in rare cases mean that CodeArtifact does not have up\-to\-date information on which packages and package versions are present in an external repository\. In this case, CodeArtifact might permit a package version to be published that it would normally deny\.