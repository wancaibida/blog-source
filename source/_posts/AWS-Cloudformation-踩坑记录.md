title: AWS Cloudformation 踩坑记录
author: 大丈夫没问题
tags:
  - aws
  - cloudformation
  - mustache
categories:
  - linux
date: 2019-06-20 20:43:00
---
最近用cloudformation时遇到了一些坑，在此记录一下。

## Conditions变量无法直接当布尔类型使用

[Conditions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html)可以动态控制一些资源的创建：

```
Conditions: 
  CreateProdResources: !Equals [ !Ref EnvType, prod ]
Resources: 
  EC2Instance: 
    Type: "AWS::EC2::Instance"
    Properties: 
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
  MountPoint: 
    Type: "AWS::EC2::VolumeAttachment"
    Condition: CreateProdResources
    Properties: 
      InstanceId: 
        !Ref EC2Instance
      VolumeId: 
        !Ref NewVolume
      Device: /dev/sdh
  NewVolume: 
    Type: "AWS::EC2::Volume"
    Condition: CreateProdResources
    Properties: 
      Size: 100
      AvailabilityZone: 
        !GetAtt EC2Instance.AvailabilityZone
```

但是如果在其他地方像这样使用会报`Unresolved resource dependencies CreateResources`错误

```
            /xxx/xx/xx.conf:
              source:
                !Sub https://${bucket}.s3.amazonaws.com/${prefix}/xx.conf
              context:
                create_resources: !Ref CreateResources
              mode: "000644"
              owner: root
              group: root
```
解决办法就时将`create_resources: !Ref CreateResources` 改成 `create_resources: !If [CreateResources, "true output ", "false output"]`

## aws-resource-init-files 传递布尔变量

AWS::CloudFormation::Init中 [files](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-init.html#aws-resource-init-files)属性可以用来创建文件，`files`的`context`属性可以传递变量，来动态生成文件内容.
AWS文档里说文件是通过类似[mustache](http://mustache.github.io/mustache.5.html)的方式来处理的：

```
A typical Mustache template:

Hello {{name}}
You have just won {{value}} dollars!
{{#in_ca}}
Well, {{taxed_value}} dollars, after taxes.
{{/in_ca}}
Given the following hash:

{
  "name": "Chris",
  "value": 10000,
  "taxed_value": 10000 - (10000 * 0.4),
  "in_ca": true
}
Will produce the following:

Hello Chris
You have just won 10000 dollars!
Well, 6000.0 dollars, after taxes.
```

这里有个坑就是布尔类型在cloudformation里时当做字符类型来传递的，AWS官方认为这是一个feature而不是bug: [What you have observed is a known behaviour with the CloudFormation servie](https://forums.aws.amazon.com/thread.jspa?messageID=898706&#898706)
例如：

xxx.conf

```
{{#create_resources}}
this is true
{{/create_resources}}

{{^create_resources}}
this is false
{{/create_resources}}
```

yaml配置文件

```
            /xxx.conf:
              authentication: S3BucketAccess
              source:
                !Sub https://${bucket}.s3.amazonaws.com/${prefix}/xxx.conf
              context:
                create_resources: !If [CreateResources, true, false]
              mode: "000644"
              owner: root
              group: root
```

这里无论 `CreateResources` 是 `true` 还是 `false`，文件内容始终会是 `this is true`，因为配置文件里的 `true` 和 `false` 都被转成了字符类型。

这里的解决办法比较hack，就是将`!If [CreateResources, true, false]`改成`!If [CreateResources, [1], []]`:

```
            /xxx.conf:
              authentication: S3BucketAccess
              source:
                !Sub https://${bucket}.s3.amazonaws.com/${prefix}/xxx.conf
              context:
                create_resources: !If [CreateResources, [1], []]
              mode: "000644"
              owner: root
              group: root
```

这里主要参考了 `mustache` 文档里 `Inverted Sections` : [https://mustache.github.io/mustache.5.html](https://mustache.github.io/mustache.5.html)


## aws-resource-init 资源更新问题
[aws-resource-init-sources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-init.html#aws-resource-init-sources)可以用来下载压缩文件并解压到EC2指定目录，但这里有个问题就是如果目标文件夹已存在，cloudformation会将两文件夹内容合并而不是替换文件夹！比如AWS部署的项目使用的依赖`LibraryA 1.0`，将library升级到`Library A 1.1`并通过cloudformation source下载并解压新本版的项目到目标目录后，目标目录的项目文件夹里会同时存在`Library A`的`1.0`和`1.1`版本。

解决办法也很简单，在用sources下载前将目标文件夹移除。

AWS平台的服务总有一些奇怪的限制或bug，调试时总会花不少时间，在此记录一下，希望能帮助到遇到相同问题的人，节约大家时间。

## Update
* 2019-07-07 添加aws-resource-init资源更新问题