# Delete a domain<a name="delete-domain"></a>

You can delete a domain using the CodeArtifact console or the AWS Command Line Interface \(AWS CLI\)\. You can't delete a domain that contains repositories\. Before you delete the domain, you must first delete its repositories\. For more information, see [Delete a repository](delete-repo.md)\.

**Topics**
+ [Delete a domain \(console\)](#delete-domain-console)
+ [Delete a domain \(AWS CLI\)](#delete-domain-cli)

## Delete a domain \(console\)<a name="delete-domain-console"></a>

1. Open the AWS CodeArtifact console at [https://console\.aws\.amazon\.com/codesuite/codeartifact/home](https://console.aws.amazon.com/codesuite/codeartifact/home)\.

1.  In the navigation pane, choose **Domains**, then choose the domain that you want to delete\. 

1.  Choose **Delete**\. 

## Delete a domain \(AWS CLI\)<a name="delete-domain-cli"></a>

Use the delete\-domain command to delete a domain\. 

```
aws codeartifact delete-domain --domain my_domain --domain-owner 111122223333
```

 JSON\-formatted data appears in the output with details about the deleted domain\.

```
{
    "domain": {
        "name": "my_domain",
        "owner": "111122223333",
        "arn": "arn:aws:codeartifact:us-west-2:111122223333:domain/my_domain",
        "status": "Active",
        "encryptionKey": "arn:aws:kms:us-west-2:111122223333:key/your-kms-key",
        "repositoryCount": 0,
        "assetSizeBytes": 0,
        "createdTime": "2020-10-12T16:51:18.039000-04:00"
    }
}
```