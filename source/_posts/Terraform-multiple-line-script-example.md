title: Terraform multiple line script example
author: 大丈夫没问题
tags:
  - terraform
categories:
  - linux
date: 2021-03-25 21:23:00
---
```
resource "null_resource" "cluster" {

  connection {
    type        = "ssh"
    user        = "admin"
    private_key = base64decode(var.private_key)
    host        = aws_lightsail_instance.lightsail.public_ip_address
    timeout     = "3600s"
  }

  provisioner "local-exec" {
    command = <<EOT
            echo "{\"host\":\"${random_string.subdomain.result}\",\"type\":\"A\",\"answer\":\"${aws_lightsail_instance.lightsail.public_ip_address}\",\"ttl\":300}" | \
            curl "https://api.name.com/v4/domains/${var.name_domain}/records" \
            -s \
            -X POST \
            -H "Accept: application/json" \
            -H "Content-Type: application/json" \
            -u "${var.name_username}":"${var.name_token}" \
            -d @-
        EOT
  }

}
```