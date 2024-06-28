---
title: "Minimum IAM Permissions (for system admins)"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 4
---

You can run this workshop with the default Administrator role.

If you need minimum permissions enabled for participants, here is the required policy JSON:

{{% notice info %}}
As an administrator, you will need to create one Cloud9 environment in the account to automatically create some required service roles needed for the workshop users. See [AWS Cloud9 setup instructions](https://catalog.workshops.aws/building-with-amazon-bedrock/en-US/prerequisites/cloud9-setup) here. You can then immediately delete the Cloud9 environment you created as an administrator.
{{% /notice %}}

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"cloud9:CreateEnvironmentSSH",
				"cloud9:CreateEnvironmentEC2"
			],
			"Resource": "*",
			"Condition": {
				"Null": {
					"cloud9:OwnerArn": "true"
				}
			}
		},
		{
			"Sid": "VisualEditor1",
			"Effect": "Allow",
			"Action": "cloud9:GetUserPublicKey",
			"Resource": "*",
			"Condition": {
				"Null": {
					"cloud9:UserArn": "true"
				}
			}
		},
		{
			"Sid": "VisualEditor2",
			"Effect": "Allow",
			"Action": "cloud9:DescribeEnvironmentMemberships",
			"Resource": "*",
			"Condition": {
				"Null": {
					"cloud9:UserArn": "true",
					"cloud9:EnvironmentId": "true"
				}
			}
		},
		{
			"Sid": "VisualEditor3",
			"Effect": "Allow",
			"Action": [
				"ssm:GetConnectionStatus",
				"cloud9:TagResource",
				"cloud9:GetUserSettings",
				"iam:ListRoles",
				"ssm:StartSession",
				"iam:ListInstanceProfilesForRole",
				"cloud9:ListEnvironments",
				"cloud9:DescribeEnvironments",
				"cloud9:ListTagsForResource",
				"ec2:DescribeInstanceTypeOfferings",
				"ec2:DescribeVpcs",
				"iam:ListUsers",
				"iam:GetUser",
				"cloud9:UpdateUserSettings",
				"ec2:DescribeSubnets",
				"ec2:DescribeRouteTables"
			],
			"Resource": "*"
		},
		{
			"Sid": "VisualEditor4",
			"Effect": "Allow",
			"Action": "iam:CreateServiceLinkedRole",
			"Resource": "*",
			"Condition": {
				"StringLike": {
					"iam:AWSServiceName": "cloud9.amazonaws.com"
				}
			}
		},
		{
			"Sid": "VisualEditor5",
			"Effect": "Allow",
			"Action": [
				"ssm:GetConnectionStatus",
				"ssm:StartSession"
			],
			"Resource": "arn:aws:ec2:::instance/*",
			"Condition": {
				"StringEquals": {
					"aws:CalledViaFirst": "cloud9.amazonaws.com"
				},
				"StringLike": {
					"ssm:resourceTag/aws:cloud9:environment": ""
				}
			}
		},
		{
			"Sid": "VisualEditor6",
			"Effect": "Allow",
			"Action": "ssm:StartSession",
			"Resource": [
				"*"
			]
		},
		{
			"Sid": "VisualEditor7",
			"Effect": "Allow",
			"Action": [
				"bedrock:GetFoundationModelAvailability",
				"bedrock:ListCustomModels",
				"bedrock:ListProvisionedModelThroughputs",
				"bedrock:ListFoundationModels",
				"bedrock:InvokeModel",
				"bedrock:InvokeModelWithResponseStream",
				"bedrock:PutFoundationModelEntitlement",
				"bedrock:ListFoundationModelAgreementOffers",
				"bedrock:PutUseCaseForModelAccess",
				"bedrock:GetUseCaseForModelAccess",
				"bedrock:CreateFoundationModelAgreement",
				"bedrock:ApplyGuardrail",
				"bedrock:CreateGuardrail",
				"bedrock:CreateGuardrailVersion",
				"bedrock:DeleteGuardrail",
				"bedrock:GetGuardrail",
				"bedrock:ListGuardrails",
				"bedrock:UpdateGuardrail",
				"bedrock:ListAgents",
				"bedrock:ListTagsForResource",
				"bedrock:ListKnowledgeBases",
				"bedrock:RetrieveAndGenerate",
				"bedrock:DetectGeneratedContent"
			],
			"Resource": "*"
		}
	]
}
```