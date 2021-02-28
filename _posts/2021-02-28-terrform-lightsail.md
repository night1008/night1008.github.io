---
layout: post
keywords: terrform lightsail
description: 使用terrform管理lightsail
title: 使用terrform管理lightsail
comments: true
---

同事强烈推荐使用terrform管理机器，目前有在使用aws上的lightsail服务，
就拿这个学习如何使用terrform。

电脑环境上mac，需要在aws上添加用户和密钥，
见教程[cli-configure-quickstart](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html),

然后再电脑上安装aws的命令行工具，```brew install awscli```

然后执行 ```aws configure``` 设置密钥(位置 **～/.aws/credentials**)， 内容如下

```
[default]
aws_access_key_id = xxx
aws_secret_access_key = xxx
```

添加 **main.tf**，内容如下，

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "ap-northeast-2"
}

resource "aws_lightsail_instance" "gitlab_test" {
  name              = "custom_gitlab"
  availability_zone = "ap-northeast-2a"
  blueprint_id      = "ubuntu_20_04"
  bundle_id         = "nano_2_0"
  #   key_pair_name     = "some_key_name"
  tags = {
    foo = "bar"
  }
}

```

最后执行以下命令可以进行创建和删除

```sh
terrform init
terrform plan
terrform apply
terrform destroy
```


另外执行可以查看区域可用服务

```
ENV AWS_DEFAULT_REGION=ap-northeast-1 aws lightsail get-blueprints
```

