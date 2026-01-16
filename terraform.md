# Terraform note
## file upload
``` 
provisioner "file" {
	source = "app.conf"
   destination = "/etc/myapp.conf"
   
   connect {
   	user="${var.instance_username}"
    password="${var.instance_password}"
   }
}
```

## SSH Key pair
typically, you'll use SSH key pair
``` 
resource "aws_key_pair" "edward_key" {
	key_name="mykey"
   public_key="ssh_rsa my_public_key"
}
```

## remote exec

``` 
provisioner "remote-exec" {
	Inline [
   	"chmod +x /opt/script.sh",
    "sudo /opt/script.sh arguments"
   ]
}
```

