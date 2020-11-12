# Serverless Chat

A serverless web chat built using AWS Lambda, AWS IoT (for WebSockets) and Amazon DynamoDB.  It includes Amazon Translate integration to allow each particants to read and write messages in their chosen language.

The architecture of this application is described in this article:

- [Serverless beyond Functions](https://medium.com/danilop/serverless-beyond-functions-cd81ee4c6b8d)

## Deploying to AWS

A script is provided, `deploy.sh` which uses AWS CloudFormation to provision all the resources needed for this demo. To use it:

- Create an AWS account.
- Visit the [IoT management page](https://console.aws.amazon.com/iot/home) in the AWS web console and ensure that an IoT endpoint has been provisioned for your account.
- Install the [AWS command line tools](https://aws.amazon.com/cli/) and set up your credentials.
- Run the `deploy.sh` script, specifying a name for your new CloudFormation stack, an AWS region and the name of an S3 bucket where the CloudFormation config files will be stored. The S3 bucket will be created if it does not exist.

  ./deploy.sh LambdaChatStack us-west-1 my.s3.bucket.name

Once the AWS resources have been provisioned, the script will print a URL to visit in your browser to see the demo.

NB: The Kinesis functionality has been disabled because it is billed per shard-hour. To enable it, edit `cloudformation/template.yaml` and uncomment the relevant lines before running `deploy.sh`.

## Amazon Translate

Amazon Translate will translate messages between recipients.  It will take the specified language of the caller, as defined via a dropdown on their client UI, and attach that to the message sent to all other chat members.  Translate will be passed that language code, and the preferred language code of the recipient, and will translate the text.  Note, if a participant enters text in a different language then the results from Translate will not be as expected - participants should stick to their chosen language.

#### Amazon Translate - Custom Terminology

A Custom Terminology can now be defined per participant, and this is sent along with each client message.  Prior to translation, the terminology name will be remapped:

`user-terminology-name` + `-` +  `source-lang`

If this Custom Terminology exists then it will be used, and ignored if not.  This allows each particpant in the chat to use the same terminology base name, and then the correct language variant is used by Translate.  In the following example, we have two participants, and the German-speaking participant sends a message to their English-speaking colleague:

- German Participant
  - Preferred language = `de`
  - Defined terminology = `tech-terms`
- English Participant
  - Preferred language = `en`
  - Terminology used by Translate for messages from German Participant = `tech-terms-de`

It does not matter if the recipient of the message has defined a terminology or not, as it the message sender that specifies the terminology to be used to translate their message.  Other participants could have other terminologies defined, but it's likely that they will either be the same for all participants, or simply undefined for some.

On the configuration side, you will need to create Custom Terminology file for each language that you want to translate from, which is standard for this feature of Amazon Translate.  In this example you would define two, one called `tech-terms-de` and another called `tech-terms-en`, and users just need to configure that base-term in their UI - of course, you could have multiple sets of terminologies defined for different use cases if you wish.