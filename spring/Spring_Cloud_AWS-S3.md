# Spring Cloud AWS - S3

- 레퍼런스
    - [https://github.com/aws/aws-sdk-java](https://github.com/aws/aws-sdk-java)
    - [https://github.com/spring-cloud/spring-cloud-aws](https://github.com/spring-cloud/spring-cloud-aws)
    - [https://github.com/awspring/spring-cloud-aws](https://github.com/awspring/spring-cloud-aws)
- 레퍼런스: [https://spring.io/blog/2021/03/17/spring-cloud-aws-2-3-is-now-available](https://spring.io/blog/2021/03/17/spring-cloud-aws-2-3-is-now-available)

### `com.amazonaws:aws-java-sdk`

---

- `build.gradle.kts`

    ```yaml
    // aws
    implementation("com.amazonaws", "aws-java-sdk", "1.12.37")
    ```

- 위의 의존성 추가시 [https://github.com/aws/aws-sdk-java](https://github.com/aws/aws-sdk-java) 하위의 온갖 의존성이 다 추가된다.
    - com.amazonaws:aws-java-sdk-accessanalyzer:1.12.37
    - com.amazonaws:aws-java-sdk-acm:1.12.37
    - com.amazonaws:aws-java-sdk-acmpca:1.12.37
    - com.amazonaws:aws-java-sdk-alexaforbusiness:1.12.37
    - com.amazonaws:aws-java-sdk-amplify:1.12.37
    - com.amazonaws:aws-java-sdk-amplifybackend:1.12.37
    - com.amazonaws:aws-java-sdk-api-gateway:1.12.37
    - com.amazonaws:aws-java-sdk-apigatewaymanagementapi:1.12.37
    - com.amazonaws:aws-java-sdk-apigatewayv2:1.12.37
    - com.amazonaws:aws-java-sdk-appconfig:1.12.37
    - com.amazonaws:aws-java-sdk-appflow:1.12.37
    - com.amazonaws:aws-java-sdk-appintegrations:1.12.37
    - com.amazonaws:aws-java-sdk-applicationautoscaling:1.12.37
    - com.amazonaws:aws-java-sdk-applicationcostprofiler:1.12.37
    - com.amazonaws:aws-java-sdk-applicationinsights:1.12.37
    - com.amazonaws:aws-java-sdk-appmesh:1.12.37
    - com.amazonaws:aws-java-sdk-appregistry:1.12.37
    - com.amazonaws:aws-java-sdk-apprunner:1.12.37
    - com.amazonaws:aws-java-sdk-appstream:1.12.37
    - com.amazonaws:aws-java-sdk-appsync:1.12.37
    - com.amazonaws:aws-java-sdk-athena:1.12.37
    - com.amazonaws:aws-java-sdk-auditmanager:1.12.37
    - com.amazonaws:aws-java-sdk-augmentedairuntime:1.12.37
    - com.amazonaws:aws-java-sdk-autoscaling:1.12.37
    - com.amazonaws:aws-java-sdk-autoscalingplans:1.12.37
    - com.amazonaws:aws-java-sdk-backup:1.12.37
    - com.amazonaws:aws-java-sdk-batch:1.12.37
    - com.amazonaws:aws-java-sdk-braket:1.12.37
    - com.amazonaws:aws-java-sdk-budgets:1.12.37
    - com.amazonaws:aws-java-sdk-chime:1.12.37
    - com.amazonaws:aws-java-sdk-cloud9:1.12.37
    - com.amazonaws:aws-java-sdk-clouddirectory:1.12.37
    - com.amazonaws:aws-java-sdk-cloudformation:1.12.37
    - com.amazonaws:aws-java-sdk-cloudfront:1.12.37
    - com.amazonaws:aws-java-sdk-cloudhsm:1.12.37
    - com.amazonaws:aws-java-sdk-cloudhsmv2:1.12.37
    - com.amazonaws:aws-java-sdk-cloudsearch:1.12.37
    - com.amazonaws:aws-java-sdk-cloudtrail:1.12.37
    - com.amazonaws:aws-java-sdk-cloudwatch:1.12.37
    - com.amazonaws:aws-java-sdk-cloudwatchmetrics:1.12.37
    - com.amazonaws:aws-java-sdk-codeartifact:1.12.37
    - com.amazonaws:aws-java-sdk-codebuild:1.12.37
    - com.amazonaws:aws-java-sdk-codecommit:1.12.37
    - com.amazonaws:aws-java-sdk-codedeploy:1.12.37
    - com.amazonaws:aws-java-sdk-codeguruprofiler:1.12.37
    - com.amazonaws:aws-java-sdk-codegurureviewer:1.12.37
    - com.amazonaws:aws-java-sdk-codepipeline:1.12.37
    - com.amazonaws:aws-java-sdk-codestar:1.12.37
    - com.amazonaws:aws-java-sdk-codestarconnections:1.12.37
    - com.amazonaws:aws-java-sdk-codestarnotifications:1.12.37
    - com.amazonaws:aws-java-sdk-cognitoidentity:1.12.37
    - com.amazonaws:aws-java-sdk-cognitoidp:1.12.37
    - com.amazonaws:aws-java-sdk-cognitosync:1.12.37
    - com.amazonaws:aws-java-sdk-comprehend:1.12.37
    - com.amazonaws:aws-java-sdk-comprehendmedical:1.12.37
    - com.amazonaws:aws-java-sdk-computeoptimizer:1.12.37
    - com.amazonaws:aws-java-sdk-config:1.12.37
    - com.amazonaws:aws-java-sdk-connect:1.12.37
    - com.amazonaws:aws-java-sdk-connectcontactlens:1.12.37
    - com.amazonaws:aws-java-sdk-connectparticipant:1.12.37
    - com.amazonaws:aws-java-sdk-core:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-costandusagereport:1.12.37
    - com.amazonaws:aws-java-sdk-costexplorer:1.12.37
    - com.amazonaws:aws-java-sdk-customerprofiles:1.12.37
    - com.amazonaws:aws-java-sdk-dataexchange:1.12.37
    - com.amazonaws:aws-java-sdk-datapipeline:1.12.37
    - com.amazonaws:aws-java-sdk-datasync:1.12.37
    - com.amazonaws:aws-java-sdk-dax:1.12.37
    - com.amazonaws:aws-java-sdk-detective:1.12.37
    - com.amazonaws:aws-java-sdk-devicefarm:1.12.37
    - com.amazonaws:aws-java-sdk-devopsguru:1.12.37
    - com.amazonaws:aws-java-sdk-directconnect:1.12.37
    - com.amazonaws:aws-java-sdk-directory:1.12.37
    - com.amazonaws:aws-java-sdk-discovery:1.12.37
    - com.amazonaws:aws-java-sdk-dlm:1.12.37
    - com.amazonaws:aws-java-sdk-dms:1.12.37
    - com.amazonaws:aws-java-sdk-docdb:1.12.37
    - com.amazonaws:aws-java-sdk-dynamodb:1.12.37
    - com.amazonaws:aws-java-sdk-ebs:1.12.37
    - com.amazonaws:aws-java-sdk-ec2:1.12.37
    - com.amazonaws:aws-java-sdk-ec2instanceconnect:1.12.37
    - com.amazonaws:aws-java-sdk-ecr:1.12.37
    - com.amazonaws:aws-java-sdk-ecrpublic:1.12.37
    - com.amazonaws:aws-java-sdk-ecs:1.12.37
    - com.amazonaws:aws-java-sdk-efs:1.12.37
    - com.amazonaws:aws-java-sdk-eks:1.12.37
    - com.amazonaws:aws-java-sdk-elasticache:1.12.37
    - com.amazonaws:aws-java-sdk-elasticbeanstalk:1.12.37
    - com.amazonaws:aws-java-sdk-elasticinference:1.12.37
    - com.amazonaws:aws-java-sdk-elasticloadbalancing:1.12.37
    - com.amazonaws:aws-java-sdk-elasticloadbalancingv2:1.12.37
    - com.amazonaws:aws-java-sdk-elasticsearch:1.12.37
    - com.amazonaws:aws-java-sdk-elastictranscoder:1.12.37
    - com.amazonaws:aws-java-sdk-emr:1.12.37
    - com.amazonaws:aws-java-sdk-emrcontainers:1.12.37
    - com.amazonaws:aws-java-sdk-eventbridge:1.12.37
    - com.amazonaws:aws-java-sdk-events:1.12.37
    - com.amazonaws:aws-java-sdk-finspace:1.12.37
    - com.amazonaws:aws-java-sdk-finspacedata:1.12.37
    - com.amazonaws:aws-java-sdk-fis:1.12.37
    - com.amazonaws:aws-java-sdk-fms:1.12.37
    - com.amazonaws:aws-java-sdk-forecast:1.12.37
    - com.amazonaws:aws-java-sdk-forecastquery:1.12.37
    - com.amazonaws:aws-java-sdk-frauddetector:1.12.37
    - com.amazonaws:aws-java-sdk-fsx:1.12.37
    - com.amazonaws:aws-java-sdk-gamelift:1.12.37
    - com.amazonaws:aws-java-sdk-glacier:1.12.37
    - com.amazonaws:aws-java-sdk-globalaccelerator:1.12.37
    - com.amazonaws:aws-java-sdk-glue:1.12.37
    - com.amazonaws:aws-java-sdk-gluedatabrew:1.12.37
    - com.amazonaws:aws-java-sdk-greengrass:1.12.37
    - com.amazonaws:aws-java-sdk-greengrassv2:1.12.37
    - com.amazonaws:aws-java-sdk-groundstation:1.12.37
    - com.amazonaws:aws-java-sdk-guardduty:1.12.37
    - com.amazonaws:aws-java-sdk-health:1.12.37
    - com.amazonaws:aws-java-sdk-healthlake:1.12.37
    - com.amazonaws:aws-java-sdk-honeycode:1.12.37
    - com.amazonaws:aws-java-sdk-iam:1.12.37
    - com.amazonaws:aws-java-sdk-identitystore:1.12.37
    - com.amazonaws:aws-java-sdk-imagebuilder:1.12.37
    - com.amazonaws:aws-java-sdk-importexport:1.12.37
    - com.amazonaws:aws-java-sdk-inspector:1.12.37
    - com.amazonaws:aws-java-sdk-iot1clickdevices:1.12.37
    - com.amazonaws:aws-java-sdk-iot1clickprojects:1.12.37
    - com.amazonaws:aws-java-sdk-iot:1.12.37
    - com.amazonaws:aws-java-sdk-iotanalytics:1.12.37
    - com.amazonaws:aws-java-sdk-iotdeviceadvisor:1.12.37
    - com.amazonaws:aws-java-sdk-iotevents:1.12.37
    - com.amazonaws:aws-java-sdk-ioteventsdata:1.12.37
    - com.amazonaws:aws-java-sdk-iotfleethub:1.12.37
    - com.amazonaws:aws-java-sdk-iotjobsdataplane:1.12.37
    - com.amazonaws:aws-java-sdk-iotsecuretunneling:1.12.37
    - com.amazonaws:aws-java-sdk-iotsitewise:1.12.37
    - com.amazonaws:aws-java-sdk-iotthingsgraph:1.12.37
    - com.amazonaws:aws-java-sdk-iotwireless:1.12.37
    - com.amazonaws:aws-java-sdk-ivs:1.12.37
    - com.amazonaws:aws-java-sdk-kafka:1.12.37
    - com.amazonaws:aws-java-sdk-kendra:1.12.37
    - com.amazonaws:aws-java-sdk-kinesis:1.12.37
    - com.amazonaws:aws-java-sdk-kinesisanalyticsv2:1.12.37
    - com.amazonaws:aws-java-sdk-kinesisvideo:1.12.37
    - com.amazonaws:aws-java-sdk-kinesisvideosignalingchannels:1.12.37
    - com.amazonaws:aws-java-sdk-kms:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-lakeformation:1.12.37
    - com.amazonaws:aws-java-sdk-lambda:1.12.37
    - com.amazonaws:aws-java-sdk-lex:1.12.37
    - com.amazonaws:aws-java-sdk-lexmodelbuilding:1.12.37
    - com.amazonaws:aws-java-sdk-lexmodelsv2:1.12.37
    - com.amazonaws:aws-java-sdk-lexruntimev2:1.12.37
    - com.amazonaws:aws-java-sdk-licensemanager:1.12.37
    - com.amazonaws:aws-java-sdk-lightsail:1.12.37
    - com.amazonaws:aws-java-sdk-location:1.12.37
    - com.amazonaws:aws-java-sdk-logs:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutequipment:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutforvision:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutmetrics:1.12.37
    - com.amazonaws:aws-java-sdk-machinelearning:1.12.37
    - com.amazonaws:aws-java-sdk-macie2:1.12.37
    - com.amazonaws:aws-java-sdk-macie:1.12.37
    - com.amazonaws:aws-java-sdk-managedblockchain:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacecatalog:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacecommerceanalytics:1.12.37
    - com.amazonaws:aws-java-sdk-marketplaceentitlement:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacemeteringservice:1.12.37
    - com.amazonaws:aws-java-sdk-mechanicalturkrequester:1.12.37
    - com.amazonaws:aws-java-sdk-mediaconnect:1.12.37
    - com.amazonaws:aws-java-sdk-mediaconvert:1.12.37
    - com.amazonaws:aws-java-sdk-medialive:1.12.37
    - com.amazonaws:aws-java-sdk-mediapackage:1.12.37
    - com.amazonaws:aws-java-sdk-mediapackagevod:1.12.37
    - com.amazonaws:aws-java-sdk-mediastore:1.12.37
    - com.amazonaws:aws-java-sdk-mediastoredata:1.12.37
    - com.amazonaws:aws-java-sdk-mediatailor:1.12.37
    - com.amazonaws:aws-java-sdk-mgn:1.12.37
    - com.amazonaws:aws-java-sdk-migrationhub:1.12.37
    - com.amazonaws:aws-java-sdk-migrationhubconfig:1.12.37
    - com.amazonaws:aws-java-sdk-mobile:1.12.37
    - com.amazonaws:aws-java-sdk-models:1.12.37
    - com.amazonaws:aws-java-sdk-mq:1.12.37
    - com.amazonaws:aws-java-sdk-mwaa:1.12.37
    - com.amazonaws:aws-java-sdk-neptune:1.12.37
    - com.amazonaws:aws-java-sdk-networkfirewall:1.12.37
    - com.amazonaws:aws-java-sdk-networkmanager:1.12.37
    - com.amazonaws:aws-java-sdk-nimblestudio:1.12.37
    - com.amazonaws:aws-java-sdk-opsworks:1.12.37
    - com.amazonaws:aws-java-sdk-opsworkscm:1.12.37
    - com.amazonaws:aws-java-sdk-organizations:1.12.37
    - com.amazonaws:aws-java-sdk-outposts:1.12.37
    - com.amazonaws:aws-java-sdk-personalize:1.12.37
    - com.amazonaws:aws-java-sdk-personalizeevents:1.12.37
    - com.amazonaws:aws-java-sdk-personalizeruntime:1.12.37
    - com.amazonaws:aws-java-sdk-pi:1.12.37
    - com.amazonaws:aws-java-sdk-pinpoint:1.12.37
    - com.amazonaws:aws-java-sdk-pinpointemail:1.12.37
    - com.amazonaws:aws-java-sdk-pinpointsmsvoice:1.12.37
    - com.amazonaws:aws-java-sdk-polly:1.12.37
    - com.amazonaws:aws-java-sdk-pricing:1.12.37
    - com.amazonaws:aws-java-sdk-prometheus:1.12.37
    - com.amazonaws:aws-java-sdk-proton:1.12.37
    - com.amazonaws:aws-java-sdk-qldb:1.12.37
    - com.amazonaws:aws-java-sdk-qldbsession:1.12.37
    - com.amazonaws:aws-java-sdk-quicksight:1.12.37
    - com.amazonaws:aws-java-sdk-ram:1.12.37
    - com.amazonaws:aws-java-sdk-rds:1.12.37
    - com.amazonaws:aws-java-sdk-rdsdata:1.12.37
    - com.amazonaws:aws-java-sdk-redshift:1.12.37
    - com.amazonaws:aws-java-sdk-redshiftdataapi:1.12.37
    - com.amazonaws:aws-java-sdk-rekognition:1.12.37
    - com.amazonaws:aws-java-sdk-resourcegroups:1.12.37
    - com.amazonaws:aws-java-sdk-resourcegroupstaggingapi:1.12.37
    - com.amazonaws:aws-java-sdk-robomaker:1.12.37
    - com.amazonaws:aws-java-sdk-route53:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoverycluster:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoverycontrolconfig:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoveryreadiness:1.12.37
    - com.amazonaws:aws-java-sdk-route53resolver:1.12.37
    - com.amazonaws:aws-java-sdk-s3:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-s3control:1.12.37
    - com.amazonaws:aws-java-sdk-s3outposts:1.12.37
    - com.amazonaws:aws-java-sdk-sagemaker:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakeredgemanager:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakerfeaturestoreruntime:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakerruntime:1.12.37
    - com.amazonaws:aws-java-sdk-savingsplans:1.12.37
    - com.amazonaws:aws-java-sdk-schemas:1.12.37
    - com.amazonaws:aws-java-sdk-secretsmanager:1.12.37
    - com.amazonaws:aws-java-sdk-securityhub:1.12.37
    - com.amazonaws:aws-java-sdk-serverlessapplicationrepository:1.12.37
    - com.amazonaws:aws-java-sdk-servermigration:1.12.37
    - com.amazonaws:aws-java-sdk-servicecatalog:1.12.37
    - com.amazonaws:aws-java-sdk-servicediscovery:1.12.37
    - com.amazonaws:aws-java-sdk-servicequotas:1.12.37
    - com.amazonaws:aws-java-sdk-ses:1.12.37
    - com.amazonaws:aws-java-sdk-sesv2:1.12.37
    - com.amazonaws:aws-java-sdk-shield:1.12.37
    - com.amazonaws:aws-java-sdk-signer:1.12.37
    - com.amazonaws:aws-java-sdk-simpledb:1.12.37
    - com.amazonaws:aws-java-sdk-simpleworkflow:1.12.37
    - com.amazonaws:aws-java-sdk-snowball:1.12.37
    - com.amazonaws:aws-java-sdk-sns:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-sqs:1.12.37
    - com.amazonaws:aws-java-sdk-ssm:1.12.37
    - com.amazonaws:aws-java-sdk-ssmcontacts:1.12.37
    - com.amazonaws:aws-java-sdk-ssmincidents:1.12.37
    - com.amazonaws:aws-java-sdk-sso:1.12.37
    - com.amazonaws:aws-java-sdk-ssoadmin:1.12.37
    - com.amazonaws:aws-java-sdk-ssooidc:1.12.37
    - com.amazonaws:aws-java-sdk-stepfunctions:1.12.37
    - com.amazonaws:aws-java-sdk-storagegateway:1.12.37
    - com.amazonaws:aws-java-sdk-sts:1.12.37
    - com.amazonaws:aws-java-sdk-support:1.12.37
    - com.amazonaws:aws-java-sdk-swf-libraries:1.11.22
    - com.amazonaws:aws-java-sdk-synthetics:1.12.37
    - com.amazonaws:aws-java-sdk-textract:1.12.37
    - com.amazonaws:aws-java-sdk-timestreamquery:1.12.37
    - com.amazonaws:aws-java-sdk-timestreamwrite:1.12.37
    - com.amazonaws:aws-java-sdk-transcribe:1.12.37
    - com.amazonaws:aws-java-sdk-transfer:1.12.37
    - com.amazonaws:aws-java-sdk-translate:1.12.37
    - com.amazonaws:aws-java-sdk-waf:1.12.37
    - com.amazonaws:aws-java-sdk-wafv2:1.12.37
    - com.amazonaws:aws-java-sdk-wellarchitected:1.12.37
    - com.amazonaws:aws-java-sdk-workdocs:1.12.37
    - com.amazonaws:aws-java-sdk-worklink:1.12.37
    - com.amazonaws:aws-java-sdk-workmail:1.12.37
    - com.amazonaws:aws-java-sdk-workmailmessageflow:1.12.37
    - com.amazonaws:aws-java-sdk-workspaces:1.12.37
    - com.amazonaws:aws-java-sdk-xray:1.12.37
    - com.amazonaws:aws-java-sdk-xray:1.12.37
    - com.amazonaws:aws-java-sdk-workspaces:1.12.37
    - com.amazonaws:aws-java-sdk-workmailmessageflow:1.12.37
    - com.amazonaws:aws-java-sdk-workmail:1.12.37
    - com.amazonaws:aws-java-sdk-worklink:1.12.37
    - com.amazonaws:aws-java-sdk-workdocs:1.12.37
    - com.amazonaws:aws-java-sdk-wellarchitected:1.12.37
    - com.amazonaws:aws-java-sdk-wafv2:1.12.37
    - com.amazonaws:aws-java-sdk-waf:1.12.37
    - com.amazonaws:aws-java-sdk-translate:1.12.37
    - com.amazonaws:aws-java-sdk-transfer:1.12.37
    - com.amazonaws:aws-java-sdk-transcribe:1.12.37
    - com.amazonaws:aws-java-sdk-timestreamwrite:1.12.37
    - com.amazonaws:aws-java-sdk-timestreamquery:1.12.37
    - com.amazonaws:aws-java-sdk-textract:1.12.37
    - com.amazonaws:aws-java-sdk-synthetics:1.12.37
    - com.amazonaws:aws-java-sdk-swf-libraries:1.11.22
    - com.amazonaws:aws-java-sdk-support:1.12.37
    - com.amazonaws:aws-java-sdk-sts:1.12.37
    - com.amazonaws:aws-java-sdk-storagegateway:1.12.37
    - com.amazonaws:aws-java-sdk-stepfunctions:1.12.37
    - com.amazonaws:aws-java-sdk-ssooidc:1.12.37
    - com.amazonaws:aws-java-sdk-ssoadmin:1.12.37
    - com.amazonaws:aws-java-sdk-sso:1.12.37
    - com.amazonaws:aws-java-sdk-ssmincidents:1.12.37
    - com.amazonaws:aws-java-sdk-ssmcontacts:1.12.37
    - com.amazonaws:aws-java-sdk-ssm:1.12.37
    - com.amazonaws:aws-java-sdk-sqs:1.12.37
    - com.amazonaws:aws-java-sdk-sns:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-snowball:1.12.37
    - com.amazonaws:aws-java-sdk-simpleworkflow:1.12.37
    - com.amazonaws:aws-java-sdk-simpledb:1.12.37
    - com.amazonaws:aws-java-sdk-signer:1.12.37
    - com.amazonaws:aws-java-sdk-shield:1.12.37
    - com.amazonaws:aws-java-sdk-sesv2:1.12.37
    - com.amazonaws:aws-java-sdk-ses:1.12.37
    - com.amazonaws:aws-java-sdk-servicequotas:1.12.37
    - com.amazonaws:aws-java-sdk-servicediscovery:1.12.37
    - com.amazonaws:aws-java-sdk-servicecatalog:1.12.37
    - com.amazonaws:aws-java-sdk-servermigration:1.12.37
    - com.amazonaws:aws-java-sdk-serverlessapplicationrepository:1.12.37
    - com.amazonaws:aws-java-sdk-securityhub:1.12.37
    - com.amazonaws:aws-java-sdk-secretsmanager:1.12.37
    - com.amazonaws:aws-java-sdk-schemas:1.12.37
    - com.amazonaws:aws-java-sdk-savingsplans:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakerruntime:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakerfeaturestoreruntime:1.12.37
    - com.amazonaws:aws-java-sdk-sagemakeredgemanager:1.12.37
    - com.amazonaws:aws-java-sdk-sagemaker:1.12.37
    - com.amazonaws:aws-java-sdk-s3outposts:1.12.37
    - com.amazonaws:aws-java-sdk-s3control:1.12.37
    - com.amazonaws:aws-java-sdk-s3:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-route53resolver:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoveryreadiness:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoverycontrolconfig:1.12.37
    - com.amazonaws:aws-java-sdk-route53recoverycluster:1.12.37
    - com.amazonaws:aws-java-sdk-route53:1.12.37
    - com.amazonaws:aws-java-sdk-robomaker:1.12.37
    - com.amazonaws:aws-java-sdk-resourcegroupstaggingapi:1.12.37
    - com.amazonaws:aws-java-sdk-resourcegroups:1.12.37
    - com.amazonaws:aws-java-sdk-rekognition:1.12.37
    - com.amazonaws:aws-java-sdk-redshiftdataapi:1.12.37
    - com.amazonaws:aws-java-sdk-redshift:1.12.37
    - com.amazonaws:aws-java-sdk-rdsdata:1.12.37
    - com.amazonaws:aws-java-sdk-rds:1.12.37
    - com.amazonaws:aws-java-sdk-ram:1.12.37
    - com.amazonaws:aws-java-sdk-quicksight:1.12.37
    - com.amazonaws:aws-java-sdk-qldbsession:1.12.37
    - com.amazonaws:aws-java-sdk-qldb:1.12.37
    - com.amazonaws:aws-java-sdk-proton:1.12.37
    - com.amazonaws:aws-java-sdk-prometheus:1.12.37
    - com.amazonaws:aws-java-sdk-pricing:1.12.37
    - com.amazonaws:aws-java-sdk-polly:1.12.37
    - com.amazonaws:aws-java-sdk-pinpointsmsvoice:1.12.37
    - com.amazonaws:aws-java-sdk-pinpointemail:1.12.37
    - com.amazonaws:aws-java-sdk-pinpoint:1.12.37
    - com.amazonaws:aws-java-sdk-pi:1.12.37
    - com.amazonaws:aws-java-sdk-personalizeruntime:1.12.37
    - com.amazonaws:aws-java-sdk-personalizeevents:1.12.37
    - com.amazonaws:aws-java-sdk-personalize:1.12.37
    - com.amazonaws:aws-java-sdk-outposts:1.12.37
    - com.amazonaws:aws-java-sdk-organizations:1.12.37
    - com.amazonaws:aws-java-sdk-opsworkscm:1.12.37
    - com.amazonaws:aws-java-sdk-opsworks:1.12.37
    - com.amazonaws:aws-java-sdk-nimblestudio:1.12.37
    - com.amazonaws:aws-java-sdk-networkmanager:1.12.37
    - com.amazonaws:aws-java-sdk-networkfirewall:1.12.37
    - com.amazonaws:aws-java-sdk-neptune:1.12.37
    - com.amazonaws:aws-java-sdk-mwaa:1.12.37
    - com.amazonaws:aws-java-sdk-mq:1.12.37
    - com.amazonaws:aws-java-sdk-models:1.12.37
    - com.amazonaws:aws-java-sdk-mobile:1.12.37
    - com.amazonaws:aws-java-sdk-migrationhubconfig:1.12.37
    - com.amazonaws:aws-java-sdk-migrationhub:1.12.37
    - com.amazonaws:aws-java-sdk-mgn:1.12.37
    - com.amazonaws:aws-java-sdk-mediatailor:1.12.37
    - com.amazonaws:aws-java-sdk-mediastoredata:1.12.37
    - com.amazonaws:aws-java-sdk-mediastore:1.12.37
    - com.amazonaws:aws-java-sdk-mediapackagevod:1.12.37
    - com.amazonaws:aws-java-sdk-mediapackage:1.12.37
    - com.amazonaws:aws-java-sdk-medialive:1.12.37
    - com.amazonaws:aws-java-sdk-mediaconvert:1.12.37
    - com.amazonaws:aws-java-sdk-mediaconnect:1.12.37
    - com.amazonaws:aws-java-sdk-mechanicalturkrequester:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacemeteringservice:1.12.37
    - com.amazonaws:aws-java-sdk-marketplaceentitlement:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacecommerceanalytics:1.12.37
    - com.amazonaws:aws-java-sdk-marketplacecatalog:1.12.37
    - com.amazonaws:aws-java-sdk-managedblockchain:1.12.37
    - com.amazonaws:aws-java-sdk-macie:1.12.37
    - com.amazonaws:aws-java-sdk-macie2:1.12.37
    - com.amazonaws:aws-java-sdk-machinelearning:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutmetrics:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutforvision:1.12.37
    - com.amazonaws:aws-java-sdk-lookoutequipment:1.12.37
    - com.amazonaws:aws-java-sdk-logs:1.12.37
    - com.amazonaws:aws-java-sdk-location:1.12.37
    - com.amazonaws:aws-java-sdk-lightsail:1.12.37
    - com.amazonaws:aws-java-sdk-licensemanager:1.12.37
    - com.amazonaws:aws-java-sdk-lexruntimev2:1.12.37
    - com.amazonaws:aws-java-sdk-lexmodelsv2:1.12.37
    - com.amazonaws:aws-java-sdk-lexmodelbuilding:1.12.37
    - com.amazonaws:aws-java-sdk-lex:1.12.37
    - com.amazonaws:aws-java-sdk-lambda:1.12.37
    - com.amazonaws:aws-java-sdk-lakeformation:1.12.37
    - com.amazonaws:aws-java-sdk-kms:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-kinesisvideosignalingchannels:1.12.37
    - com.amazonaws:aws-java-sdk-kinesisvideo:1.12.37
    - com.amazonaws:aws-java-sdk-kinesisanalyticsv2:1.12.37
    - com.amazonaws:aws-java-sdk-kinesis:1.12.37
    - com.amazonaws:aws-java-sdk-kendra:1.12.37
    - com.amazonaws:aws-java-sdk-kafka:1.12.37
    - com.amazonaws:aws-java-sdk-ivs:1.12.37
    - com.amazonaws:aws-java-sdk-iotwireless:1.12.37
    - com.amazonaws:aws-java-sdk-iotthingsgraph:1.12.37
    - com.amazonaws:aws-java-sdk-iotsitewise:1.12.37
    - com.amazonaws:aws-java-sdk-iotsecuretunneling:1.12.37
    - com.amazonaws:aws-java-sdk-iotjobsdataplane:1.12.37
    - com.amazonaws:aws-java-sdk-iotfleethub:1.12.37
    - com.amazonaws:aws-java-sdk-ioteventsdata:1.12.37
    - com.amazonaws:aws-java-sdk-iotevents:1.12.37
    - com.amazonaws:aws-java-sdk-iotdeviceadvisor:1.12.37
    - com.amazonaws:aws-java-sdk-iotanalytics:1.12.37
    - com.amazonaws:aws-java-sdk-iot:1.12.37
    - com.amazonaws:aws-java-sdk-iot1clickprojects:1.12.37
    - com.amazonaws:aws-java-sdk-iot1clickdevices:1.12.37
    - com.amazonaws:aws-java-sdk-inspector:1.12.37
    - com.amazonaws:aws-java-sdk-importexport:1.12.37
    - com.amazonaws:aws-java-sdk-imagebuilder:1.12.37
    - com.amazonaws:aws-java-sdk-identitystore:1.12.37
    - com.amazonaws:aws-java-sdk-iam:1.12.37
    - com.amazonaws:aws-java-sdk-honeycode:1.12.37
    - com.amazonaws:aws-java-sdk-healthlake:1.12.37
    - com.amazonaws:aws-java-sdk-health:1.12.37
    - com.amazonaws:aws-java-sdk-guardduty:1.12.37
    - com.amazonaws:aws-java-sdk-groundstation:1.12.37
    - com.amazonaws:aws-java-sdk-greengrassv2:1.12.37
    - com.amazonaws:aws-java-sdk-greengrass:1.12.37
    - com.amazonaws:aws-java-sdk-gluedatabrew:1.12.37
    - com.amazonaws:aws-java-sdk-glue:1.12.37
    - com.amazonaws:aws-java-sdk-globalaccelerator:1.12.37
    - com.amazonaws:aws-java-sdk-glacier:1.12.37
    - com.amazonaws:aws-java-sdk-gamelift:1.12.37
    - com.amazonaws:aws-java-sdk-fsx:1.12.37
    - com.amazonaws:aws-java-sdk-frauddetector:1.12.37
    - com.amazonaws:aws-java-sdk-forecastquery:1.12.37
    - com.amazonaws:aws-java-sdk-forecast:1.12.37
    - com.amazonaws:aws-java-sdk-fms:1.12.37
    - com.amazonaws:aws-java-sdk-fis:1.12.37
    - com.amazonaws:aws-java-sdk-finspacedata:1.12.37
    - com.amazonaws:aws-java-sdk-finspace:1.12.37
    - com.amazonaws:aws-java-sdk-events:1.12.37
    - com.amazonaws:aws-java-sdk-eventbridge:1.12.37
    - com.amazonaws:aws-java-sdk-emrcontainers:1.12.37
    - com.amazonaws:aws-java-sdk-emr:1.12.37
    - com.amazonaws:aws-java-sdk-elastictranscoder:1.12.37
    - com.amazonaws:aws-java-sdk-elasticsearch:1.12.37
    - com.amazonaws:aws-java-sdk-elasticloadbalancingv2:1.12.37
    - com.amazonaws:aws-java-sdk-elasticloadbalancing:1.12.37
    - com.amazonaws:aws-java-sdk-elasticinference:1.12.37
    - com.amazonaws:aws-java-sdk-elasticbeanstalk:1.12.37
    - com.amazonaws:aws-java-sdk-elasticache:1.12.37
    - com.amazonaws:aws-java-sdk-eks:1.12.37
    - com.amazonaws:aws-java-sdk-efs:1.12.37
    - com.amazonaws:aws-java-sdk-ecs:1.12.37
    - com.amazonaws:aws-java-sdk-ecrpublic:1.12.37
    - com.amazonaws:aws-java-sdk-ecr:1.12.37
    - com.amazonaws:aws-java-sdk-ec2instanceconnect:1.12.37
    - com.amazonaws:aws-java-sdk-ec2:1.12.37
    - com.amazonaws:aws-java-sdk-ebs:1.12.37
    - com.amazonaws:aws-java-sdk-dynamodb:1.12.37
    - com.amazonaws:aws-java-sdk-docdb:1.12.37
    - com.amazonaws:aws-java-sdk-dms:1.12.37
    - com.amazonaws:aws-java-sdk-dlm:1.12.37
    - com.amazonaws:aws-java-sdk-discovery:1.12.37
    - com.amazonaws:aws-java-sdk-directory:1.12.37
    - com.amazonaws:aws-java-sdk-directconnect:1.12.37
    - com.amazonaws:aws-java-sdk-devopsguru:1.12.37
    - com.amazonaws:aws-java-sdk-devicefarm:1.12.37
    - com.amazonaws:aws-java-sdk-detective:1.12.37
    - com.amazonaws:aws-java-sdk-dax:1.12.37
    - com.amazonaws:aws-java-sdk-datasync:1.12.37
    - com.amazonaws:aws-java-sdk-datapipeline:1.12.37
    - com.amazonaws:aws-java-sdk-dataexchange:1.12.37
    - com.amazonaws:aws-java-sdk-customerprofiles:1.12.37
    - com.amazonaws:aws-java-sdk-costexplorer:1.12.37
    - com.amazonaws:aws-java-sdk-costandusagereport:1.12.37
    - com.amazonaws:aws-java-sdk-core:1.12.37 (*)
    - com.amazonaws:aws-java-sdk-connectparticipant:1.12.37
    - com.amazonaws:aws-java-sdk-connectcontactlens:1.12.37
    - com.amazonaws:aws-java-sdk-connect:1.12.37
    - com.amazonaws:aws-java-sdk-config:1.12.37
    - com.amazonaws:aws-java-sdk-computeoptimizer:1.12.37
    - com.amazonaws:aws-java-sdk-comprehendmedical:1.12.37
    - com.amazonaws:aws-java-sdk-comprehend:1.12.37
    - com.amazonaws:aws-java-sdk-cognitosync:1.12.37
    - com.amazonaws:aws-java-sdk-cognitoidp:1.12.37
    - com.amazonaws:aws-java-sdk-cognitoidentity:1.12.37
    - com.amazonaws:aws-java-sdk-codestarnotifications:1.12.37
    - com.amazonaws:aws-java-sdk-codestarconnections:1.12.37
    - com.amazonaws:aws-java-sdk-codestar:1.12.37
    - com.amazonaws:aws-java-sdk-codepipeline:1.12.37
    - com.amazonaws:aws-java-sdk-codegurureviewer:1.12.37
    - com.amazonaws:aws-java-sdk-codeguruprofiler:1.12.37
    - com.amazonaws:aws-java-sdk-codedeploy:1.12.37
    - com.amazonaws:aws-java-sdk-codecommit:1.12.37
    - com.amazonaws:aws-java-sdk-codebuild:1.12.37
    - com.amazonaws:aws-java-sdk-codeartifact:1.12.37
    - com.amazonaws:aws-java-sdk-cloudwatchmetrics:1.12.37
    - com.amazonaws:aws-java-sdk-cloudwatch:1.12.37
    - com.amazonaws:aws-java-sdk-cloudtrail:1.12.37
    - com.amazonaws:aws-java-sdk-cloudsearch:1.12.37
    - com.amazonaws:aws-java-sdk-cloudhsmv2:1.12.37
    - com.amazonaws:aws-java-sdk-cloudhsm:1.12.37
    - com.amazonaws:aws-java-sdk-cloudfront:1.12.37
    - com.amazonaws:aws-java-sdk-cloudformation:1.12.37
    - com.amazonaws:aws-java-sdk-clouddirectory:1.12.37
    - com.amazonaws:aws-java-sdk-cloud9:1.12.37
    - com.amazonaws:aws-java-sdk-chime:1.12.37
    - com.amazonaws:aws-java-sdk-budgets:1.12.37
    - com.amazonaws:aws-java-sdk-braket:1.12.37
    - com.amazonaws:aws-java-sdk-batch:1.12.37
    - com.amazonaws:aws-java-sdk-backup:1.12.37
    - com.amazonaws:aws-java-sdk-autoscalingplans:1.12.37
    - com.amazonaws:aws-java-sdk-autoscaling:1.12.37
    - com.amazonaws:aws-java-sdk-augmentedairuntime:1.12.37
    - com.amazonaws:aws-java-sdk-auditmanager:1.12.37
    - com.amazonaws:aws-java-sdk-athena:1.12.37
    - com.amazonaws:aws-java-sdk-appsync:1.12.37
    - com.amazonaws:aws-java-sdk-appstream:1.12.37
    - com.amazonaws:aws-java-sdk-apprunner:1.12.37
    - com.amazonaws:aws-java-sdk-appregistry:1.12.37
    - com.amazonaws:aws-java-sdk-appmesh:1.12.37
    - com.amazonaws:aws-java-sdk-applicationinsights:1.12.37
    - com.amazonaws:aws-java-sdk-applicationcostprofiler:1.12.37
    - com.amazonaws:aws-java-sdk-applicationautoscaling:1.12.37
    - com.amazonaws:aws-java-sdk-appintegrations:1.12.37
    - com.amazonaws:aws-java-sdk-appflow:1.12.37
    - com.amazonaws:aws-java-sdk-appconfig:1.12.37
    - com.amazonaws:aws-java-sdk-apigatewayv2:1.12.37
    - com.amazonaws:aws-java-sdk-apigatewaymanagementapi:1.12.37
    - com.amazonaws:aws-java-sdk-api-gateway:1.12.37
    - com.amazonaws:aws-java-sdk-amplifybackend:1.12.37
    - com.amazonaws:aws-java-sdk-amplify:1.12.37
    - com.amazonaws:aws-java-sdk-alexaforbusiness:1.12.37
    - com.amazonaws:aws-java-sdk-acmpca:1.12.37
    - com.amazonaws:aws-java-sdk-acm:1.12.37
    - com.amazonaws:aws-java-sdk-accessanalyzer:1.12.37
- 의존성 급발진으로 인하여 놀란 마음을 부여잡고 s3 부분만 의존성 추가할까 하다가 다른 방법들을 생각해보기로 했다.

### `org.springframework.cloud:spring-cloud-aws`

---

- `build.gradle.kts`

    ```yaml
    // aws
    implementation("org.springframework.cloud", "spring-cloud-starter-aws", "Hoxton.SR12")
    ```

- [spring-cloud-aws-autoconfigure](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-aws-autoconfigure/2.2.1.RELEASE)를 통해, 아래에서 작성할 application.yml으로 S3 Client를 자동으로 셋팅해주는 기능이 있는데, AmazonS3Client 가 deprecated가 됨에 따라 AmazonS3을 사용하게 되었다.
- 그래서 해당 의존성은 사용하지 않고, service 단에서 직접 설정해줘야 한다.
- `application.yml`

    ```yaml
    cloud:
      aws:
        credentials:
          accessKey: YOUR_ACCESS_KEY
          secretKey: YOUR_SECRET_KEY
        s3:
          bucket: YOUR_BUCKET_NAME
        region:
          static: YOUR_REGION
        stack:
          auto: false
    ```

    - accessKey, secretKey: AWS 계정에 부여된 key 값 (IAM 계정 사용 권장)
    - s3.bucket: S3 서비스에서 생성한 버킷 이름
    - region.static: S3를 서비스할 region 명
        - 서울은 ap-northeast-2를 작성하면 됩니다.
    - stack.auto
        - Spring Cloud 실행 시, 서버 구성을 자동화하는 CloudFormation이 자동으로 실행되는데 이를 사용하지 않겠다는 설정
        - 해당 설정을 안해주면 아래의 에러가 발생합니다.

            ```java
            org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.cloud.aws.core.env.ResourceIdResolver.BEAN_NAME': Invocation of init method failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'stackResourceRegistryFactoryBean' defined in class path resource [org/springframework/cloud/aws/autoconfigure/context/ContextStackAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.cloud.aws.core.env.stack.config.StackResourceRegistryFactoryBean]: Factory method 'stackResourceRegistryFactoryBean' threw exception; nested exception is java.lang.IllegalArgumentException: No valid instance id defined
            ```

- `Controller`

    ```java
    import com.victolee.s3exam.dto.GalleryDto;
    import com.victolee.s3exam.service.GalleryService;
    import com.victolee.s3exam.service.S3Service;
    import lombok.AllArgsConstructor;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.multipart.MultipartFile;

    import java.io.IOException;

    @Controller
    @AllArgsConstructor
    public class GalleryController {
        private S3Service s3Service;
        private GalleryService galleryService;

        @GetMapping("/gallery")
        public String dispWrite() {

            return "/gallery";
        }

        @PostMapping("/gallery")
        public String execWrite(GalleryDto galleryDto, MultipartFile file) throws IOException {
            String imgPath = s3Service.upload(file);
            galleryDto.setFilePath(imgPath);

            galleryService.savePost(galleryDto);

            return "redirect:/gallery";
        }
    }
    ```

- `S3Service`

    ```java
    package com.victolee.s3exam.service;

    import com.amazonaws.auth.AWSCredentials;
    import com.amazonaws.auth.AWSStaticCredentialsProvider;
    import com.amazonaws.auth.BasicAWSCredentials;
    import com.amazonaws.services.s3.AmazonS3;
    import com.amazonaws.services.s3.AmazonS3ClientBuilder;
    import com.amazonaws.services.s3.model.CannedAccessControlList;
    import com.amazonaws.services.s3.model.PutObjectRequest;
    import lombok.NoArgsConstructor;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Service;
    import org.springframework.web.multipart.MultipartFile;

    import javax.annotation.PostConstruct;
    import java.io.IOException;

    @Service
    @NoArgsConstructor
    public class S3Service {
        private AmazonS3 s3Client;

        @Value("${cloud.aws.credentials.accessKey}")
        private String accessKey;

        @Value("${cloud.aws.credentials.secretKey}")
        private String secretKey;

        @Value("${cloud.aws.s3.bucket}")
        private String bucket;

        @Value("${cloud.aws.region.static}")
        private String region;

        @PostConstruct
        public void setS3Client() {
            AWSCredentials credentials = new BasicAWSCredentials(this.accessKey, this.secretKey);

            s3Client = AmazonS3ClientBuilder.standard()
                    .withCredentials(new AWSStaticCredentialsProvider(credentials))
                    .withRegion(this.region)
                    .build();
        }

        public String upload(MultipartFile file) throws IOException {
            String fileName = file.getOriginalFilename();

            s3Client.putObject(new PutObjectRequest(bucket, fileName, file.getInputStream(), null)
                    .withCannedAcl(CannedAccessControlList.PublicRead));
            return s3Client.getUrl(bucket, fileName).toString();
        }
    }
    ```

    - AmazonS3Client가 deprecated됨에 따라, AmazonS3ClientBuilder를 사용
    - `@Value("${cloud.aws.credentials.accessKey}")`
        - lombok 패키지가 아닌, org.springframework.beans.factory.annotation 패키지
        - 해당 값은 application.yml에서 작성한 cloud.aws.credentials.accessKey 값
    - `@PostConstruct`
        - `new BasicAWSCredentials(this.accessKey, this.secretKey);`
            - 자격증명은 accessKey, secretKey를 의미하는데, 의존성 주입 시점에는 `@Value` 어노테이션의 값이 설정되지 않아서 `@PostConstruct`를 사용했다.
        - 의존성 주입을 마친 후 초기화를 수행하는 메서드, bean이 한 번만 초기화 될 수 있게 해준다.
        - `AmazonS3ClientBuilder`를 통해 `S3 Client`를 가져와야 하는데, 자격증명을 해줘야 S3 Client를 가져올 수 있기 때문
    - `s3Client.putObject`
        - 업로드를 하기 위해 사용되는 함수. ([AWS SDK](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/upload-objects.html) 참고)
        - `.withCannedAcl(CannedAccessControlList.PublicRead));`
            - 외부에 공개할 이미지이므로, 해당 파일에 public read 권한을 추가
        - `s3Client.getUrl(bucket, fileName).toString()`
            - 업로드를 한 후, 해당 URL을 DB에 저장할 수 있도록 URL을 반환한다.

### `io.awspring.cloud:spring-cloud-aws`

---

- 부트 2.4 이상 부터 이걸로 가야한다.
- `build.gradle.kts`

    ```yaml
    extra["springCloudAWSVersion"] = "2.3.1"

    dependencies {
        // aws
        implementation("io.awspring.cloud:spring-cloud-starter-aws")
    }

    dependencyManagement {
        imports {
            mavenBom("io.awspring.cloud:spring-cloud-aws-dependencies:${property("springCloudAWSVersion")}")
        }
    }
    ```

- 참고: [https://github.com/Danny-Hoang/springsocial/blob/master/src/main/kotlin/com/example/springsocial/service/AmazonClient.java](https://github.com/Danny-Hoang/springsocial/blob/master/src/main/kotlin/com/example/springsocial/service/AmazonClient.java)

- Release train - 스프링 부트 compatibility

- 고려해야할 부분
    - OOM: [https://stackoverflow.com/questions/36201759/how-to-set-inputstream-content-length](https://stackoverflow.com/questions/36201759/how-to-set-inputstream-content-length)
    - 환경변수 지정 필요

- 참고
    - [https://victorydntmd.tistory.com/334](https://victorydntmd.tistory.com/334)
    - [https://nirajsonawane.github.io/2021/05/16/Spring-Boot-with-AWS-S3/](https://nirajsonawane.github.io/2021/05/16/Spring-Boot-with-AWS-S3/)
    - [https://willseungh0.tistory.com/78](https://willseungh0.tistory.com/78)
    - [https://akageun.github.io/2018/06/05/aws-java-s3-upload-tip.html](https://akageun.github.io/2018/06/05/aws-java-s3-upload-tip.html)
