# What is Cloud?
Cloud computing is the on-demand availability of computer system resources, especially data storage (cloud storage) and computing power, without direct active management by the user.  We can quickly spin up resources as you need them–from infrastructure services, such as compute, storage, and databases, to Internet of Things, machine learning, data lakes and analytics, and much more. Most of the Cloud Providers work on the agreement of Pay-as-we-go, which means that startups don't need a huge amount to setup their business.

# Working on Clouds
Most of the cloud uses magnificient GUI interface. However, almost all the companies don't prefer a GUI because automation isn't possible.
So to survive in this automation world they use CLI(COMMAND-LINE-INTERFACE). CLI commands can easily be scheduled and hence, things can be automated as per choice.

# What is the main issue then?
The different Cloud services uses different CLI command. Hence it becomes a tedious task for Cloud Engineers to learn CLI commands of all the services.

# Is there any Solution?
Yes. A single tool called terraform  enables you to safely and predictably create, change, and improve infrastructure without even giving the importance to different services of cloud. It is highly intelligent. The plugins makes the terraform more genious.

# What are the minimum sevices required which need to be used to perform Automation?
* EC2 [Elastic Compute Cloud]
* EBS[Elastic Block Storage]
* S3[Simple Storage Service]
* Cloudfront

# THE PROJECT
### Launching a Web Server using Terraform

#### Step 1:- configure AWS profile in the local system using cmd.

>aws configure --profile NIRBHAY  <br>
>AWS Access Key ID [****************CKMR]: <br>
>AWS Secret Access Key [****************OKDB]: <br>
>Default region name [ap-south-1]: <br>
>Default output format [json]: <br>

#### Step 2:- After configuring your system, create a key-pair or you can use existing key-pair. Launch an ec2 instance. Note that security group should have SSH enabled on port 22 & HTTP enabled on port 80.

>provider "aws" { <br>
>  region  = "ap-south-1"  <br>
>  profile = "NIRBHAY"  <br>
>}  <br>

>resource "aws_instance" "web" {  <br>
>  ami           =   "ami-052c08d70def0ac62"  <br>
>  instance_type =   "t2.micro"  <br>
>  key_name      =   "mykey3698"  <br>
>  security_groups = ["launch-wizard-1"]  <br>
>
>connection {  <br>
>    type     = "ssh"  <br>
>    user     = "ec2-user"  <br>
>    private_key = file("C:/Users/NIRBHAY/Downloads/mykey3698.pem")  <br>
>    host     = aws_instance.web.public_ip  <br>
>  }  <br>
>
> provisioner "remote-exec" {  <br>
>    inline = [  <br>
>      "sudo yum install httpd php git -y", <br>
>      "sudo systemctl restart httpd",  <br>
>      "sudo systemctl enable httpd",  <br>
>    ]  <br>
>  }  <br>
>
>  tags = {  <br>
>    Name = "RedHatWorld"  <br>
>  }  <br>
>}  <br>

#### Step 3:- Create an EBS Volume. Always launch your EBS volume in the same availability zone as EC2 as launching in different region will result in connectivity issue.. To resolve this, retrieve the availability zone of the instance and use it.

>resource "aws_ebs_volume" "add_vol" {<br>
>  availability_zone = aws_instance.web.availability_zone<br>
>  size              = 1<br>
>
>  tags={<br>
>    Name = "RedHat_ebs"<br>
>   }<br>
>}<br>

#### Step 4:-  Attach EBS volume to the instance.
>resource "aws_volume_attachment" "ebs_att" {<br>
>  device_name = "/dev/sdd"<br>
>  volume_id   = "${aws_ebs_volume.add_vol.id}"<br>
>  instance_id = "${aws_instance.web.id}"<br>
>  force_detach = true<br>
>}<br>

#### Step 5:- Retrieve the public ip of instance and store it in a file locally as it may be used later.
>resource "null_resource" "save_ip" {<br>
>  provisioner "local-exec" {<br>
>    command = "echo ${aws_instance.web.public_ip} > public_ip.txt"<br>
>  }<br>
>}<br>

#### Step 6:- Mount EBS volume to the folder /var/ww/html so that it can be deployed by the Apache Web Server.
>resource "null_resource" "mount_vol"  {
>
>    depends_on = [ <br>
>        aws_volume_attachment.ebs_att,<br>
>      ]<br>
>
>connection {<br>
>    type     = "ssh"<br>
>    user     = "ec2-user"<br>
>    private_key = file("C:/Users/NIRBHAY/Downloads/mykey3698.pem")<br>
>    host     = aws_instance.web.public_ip<br>
>  }
>
>provisioner "remote-exec" {<br>
>    inline = [<br>
>      "sudo mkfs.ext4 /dev/xvdd",<br>
>      "sudo mount /dev/xvdd /var/www/html",<br>
>      "sudo rm -rf /var/www/html/*",<br>
>      "sudo git clone https://github.com/NirbhayMaitra/MultiCloud_proj1.git /var/www/html"<br>
>    ]<br>
>  }<br>
>}<br>
