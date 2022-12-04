### AWS 

1. Identity and Access Management (IAM)

- Tạo Alias cho tài khoản AWS

--> IAM dashboard:

`https://766685199999.signin.aws.amazon.com/console` --> Customize 

- Create user group

<img src="/Note/img/aws1.png">

<img src="/Note/img/aws2.png">

* Config MFA

- Account --> My Security Credentials --> Asign MFA devices --> Continue --> Authentication (Google authentication)

* Cấu hình Password Policy

<img src="/Note/img/aws3.png">

<img src="/Note/img/aws4.png">

* Cấu hình aws configure truy cập từ máy tính cá nhân lên AWS Cloud

- Install aws cli `https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html`

- Account --> My Security Credentials --> Create access key 

```
Run command:
aws configure
vi ~/.aws/credentials
aws s3 ls
aws s3 mb s3://cloudtest
```

* AWS Management Console

--> Budget

<img src="/Note/img/aws5.png">

<img src="/Note/img/aws6.png">

* Uớc lượng chi phí sử dụng trên AWS hàng tháng hàng năm

`https://calculator.aws/#/`








