---
title: "Minimum IAM Permissions (cho quản trị viên)"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 4
---

Bạn có thể thực hành trong workshop với role Administrator mặc định.

Nếu cần kích hoạt quyền tối thiểu cho người tham gia, sao chép chính sách tối thiểu cần thiết trong JSON:

{{% notice info %}}
Với tư cách admin, bạn sẽ phải tạo môi trường Cloud9 trong tài khoản để tự động tạo các service role cần thiết cho người thực hành. Xem [hướng dẫn thiết lập AWS Cloud9](https://catalog.workshops.aws/building-with-amazon-bedrock/en-US/preconditions/cloud9-setup). Sau đó, có thể xóa ngay môi trường Cloud9 mà bạn đã tạo với tư cách admin.
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