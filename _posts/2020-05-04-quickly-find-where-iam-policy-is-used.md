---
title: "Quickly Find Where an AWS IAM Policy is Used"
date: 2022-05-03
---

Sometimes you need to know all of the places an IAM policy is being used in your AWS account. Maybe you are making a change to that policy and you want to know the scope of the impact. Other times you may need to replace the policy with a new policy. Regardless of the need, pointing and clicking your way through the AWS console is not the most efficient way you want to accomplish this. Here is a perfect opportunity to use the AWS CLI.

The AWS Command Line Interface (CLI) is a fast and simple way to programmatically get info and take action on your AWS account in a lightweight manner. Although your can also use a number of language specific SDKs to accomplish the same thing, the AWS CLI does not require the additional overhead of some programming languages. Also, the AWS CLI is a good choice for one-time tasks that need a quick turn around. 

Please reference the [AWS Command Line Interface Documentation](https://aws.amazon.com/cli/) to get the AWS CLI set up. As you can see by reviewing the [Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html), the AWS CLI has quite an extensive and granular set of commands. With the broad choice of options here, you should be able to do whatever you need from the AWS CLI.  

In this example, we want to find out all of the places that an IAM policy is being used. Let’s start with the [IAM commands](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html) and see what we can find there. The [list-entities-for-policy](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/list-entities-for-policy.html) command looks like it will do the trick. This command lists all IAM users, groups, and roles that the specified managed policy is attached to. As with most AWS CLI commands, you can see that you have several options for input parameters. These parameters help you to narrow down the scope of the actions you want to take or the results that are returned. In addition to the documented parameter options, you can also use the _--query_ option to further filter command results (more on that later). At this point you may notice that you need the _policy-arn_ for this command. You likely already know the name of policy but you need the Amazon Resource Name (ARN). Although ARNs typically follow a common naming convention, it’s probably best that you don’t just guess at this and actually look it up. How do you get the ARN for a policy? Well, the [list-policies](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/list-policies.html) can help. The _list-policies_ command lists all the managed policies that are available in your and returns meta data about the policy including the its ARN. The this command by default returns all of the policies but we only care about one in this case. Scrolling through hundreds of policies to find the one you want is not very efficient. This is where the aforementioned _--query_ option comes in handy. We can use _--query_ to filter the results based on the policy name. Here is an example:

```
aws iam list-policies --query 'Policies[?PolicyName==`yourPolicyNameHere`].Arn
```

In addition to the _--query_ filter, we are only referencing the Arn property because that is only value we are interested in. Now that we have our Policy ARN, we can go back and use that as in input for the _list-entities-for-policy_ command. Wait a minute... who wants to run two separate commands? We can use an AWS CLI command to fulfill the input parameter of a separate command by nesting the ARN lookup right within the outer command. Here is an example for our use case:


```
aws iam list-entities-for-policy --policy-arn "$(aws iam list-policies --query 'Policies[?PolicyName==`yourPolicyNameHere`].Arn' --output text)"
```

As you can see here, we are running the list-policies command inline for the _list-entities-for-policy_ command in order to get the _policy-arn_. We also added a couple of other details to make this work. Setting the _--output_ parameter of the _list-policies_ command to **text** returns the _policy-arn_ in a format that the outer command can process. 

So with a couple of quickly run commands, we can easily search through all of the IAM roles within an AWS account to see where a specific policy is being used. Next time you have to look something up in your AWS account and you don’t want to spend your day clicking through the AWS console, think about how the AWS CLI can help get you what you need.
