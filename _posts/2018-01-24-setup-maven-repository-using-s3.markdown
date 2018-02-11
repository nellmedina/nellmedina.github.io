---
layout: post
title: "Setup Maven Repository using S3"
date: 2018-01-24 20:51:00 +0800
description: Setup of Maven repository using AWS S3.
img: maven-s3.jpg
tags: [AWS, S3, Maven]
---

As the code and team grows, its good practice to have your Maven repository in the Cloud to manage jars for snapshots and releases. There are many paid versions to provide this service like Nexus or JFrog Artifactory. I choose the **cheaper** option which is using `AWS S3` bucket service.

#### Create an S3 bucket like `repo.nellmedina.com` and set its Bucket policy
{% highlight bash %}
{
    "Version": "2012-10-17",
    "Id": "Policy1516725132304",
    "Statement": [
        {
            "Sid": "Stmt1516725077245",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::repo.nellmedina.com"
        },
        {
            "Sid": "Stmt1516725128850",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::repo.nellmedina.com/*"
        }
    ]
}
{% endhighlight %}

#### Get your AWS `aws_access_key_id` and `aws_secret_access_key` to be set on your maven's `settings.xml`
{% highlight bash %}
<servers>
    <server>
        <id>s3-release</id>
        <username>${aws_access_key_id}</username>
        <password>${aws_secret_access_key}</password>
    </server>
    <server>
        <id>s3-snapshot</id>
        <username>${aws_access_key_id}</username>
        <password>${aws_secret_access_key}</password>
    </server>
</servers>
{% endhighlight %}

#### Edit the `pom.xml` of your Maven project
{% highlight bash %}
    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
            # this is for releasing
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-release-plugin</artifactId>
				<version>2.5.3</version>
				<configuration>
					<tagNameFormat>v@{project.version}.RELEASE</tagNameFormat>
				</configuration>
			</plugin>
            # this is for snapshot
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-deploy-plugin</artifactId>
				<version>2.7</version>
				<configuration>
					<uniqueVersion>false</uniqueVersion>
				</configuration>
			</plugin>
		</plugins>
		<extensions>
            # this is for pushing files to S3
			<extension>
				<groupId>org.springframework.build</groupId>
				<artifactId>aws-maven</artifactId>
				<version>5.0.0.RELEASE</version>
			</extension>
		</extensions>
	</build>

    # declare S3 locations for releases and snaphots
	<distributionManagement>
		<repository>
			<id>s3-release</id>
			<name>Release Repository for Component1</name>
			<url>s3://repo.nellmedina.com/release</url>
		</repository>
		<snapshotRepository>
			<id>s3-snapshot</id>
			<name>Snapshot Repository for Component1</name>
			<url>s3://repo.nellmedina.com/snapshot</url>
		</snapshotRepository>
	</distributionManagement>

    # an scm location like github for content management
	<scm>
		<connection>scm:git:git@github.com:nellmedina/demo1.git</connection>
		<url>scm:git:git@github.com:nellmedina/demo1.git</url>
		<developerConnection>scm:git:git@github.com:nellmedina/demo1.git</developerConnection>
		<tag>HEAD</tag>
	</scm>

    # enable or disable releases and snaphots
	<repositories>
		<repository>
			<id>s3-release</id>
			<name>S3 Release Repository for component1</name>
			<url>s3://repo.nellmedina.com/release</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>s3-snapshot</id>
			<name>Component1-s3-snapshot-repo</name>
			<url>s3://repo.nellmedina.com/snapshot</url>
			<releases>
				<enabled>false</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
{% endhighlight %}

#### Useful maven commands
- `mvn deploy`, commits code to S3 repository as snapshot
- `mvn clean install`, commits code to local repository
- `mvn release:clean`, cleans the release
- `mvn release:prepare`, checks the scm and increments snapshot
- `mvn release:perform`, deploys release to S3 without the snapshot