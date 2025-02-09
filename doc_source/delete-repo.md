# Delete a repository<a name="delete-repo"></a>

You can delete a repository using the CodeArtifact console or the AWS CLI\. After a repository has been deleted, you can no longer push packages to it or pull packages from it\. All packages in the repository become permanently unavailable and cannot be restored\. You can create a repository with the same name, but its contents will be empty\.

**Topics**
+ [Delete a repository \(console\)](#delete-repo-console)
+ [Delete a repository \(AWS CLI\)](#delete-repo-cli)

## Delete a repository \(console\)<a name="delete-repo-console"></a>

1. Open the AWS CodeArtifact console at [https://console\.aws\.amazon\.com/codesuite/codeartifact/home](https://console.aws.amazon.com/codesuite/codeartifact/home)\.

1.  On the navigation pane, choose **Repositories**, then choose the repository that you want to delete\. 

1.  Choose **Delete** and then follow the steps to delete the domain\. 

## Delete a repository \(AWS CLI\)<a name="delete-repo-cli"></a>

Use the `delete-repository` command to delete a repository\.

```
aws codeartifact delete-repository --domain my_domain --domain-owner 111122223333 --repository my_repo
```

Example output:

```
{
    "repository": {
        "name": "my_repo",
        "administratorAccount": "123456789012",
        "domainName": "my_domain",            
        "domainOwner": "123456789012",
        "arn": "arn:aws:codeartifact:region-id:123456789012:repository/my_domain/my_repo",
        "description": "My new repository",
        "upstreams": [],
        "externalConnections": []
    }
}
```