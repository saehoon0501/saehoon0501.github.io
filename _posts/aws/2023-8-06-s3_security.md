---
title: S3 Security
date: 2023-08-06 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-s3, aws-s3_security]
---

## S3 Security

S3에 저장되는 Object에 대한 Encryption에는 총 4가지 방법이 존재한다.

- Server-Side Encryption
  - Server-Side Encryption with Amazon S3-Managed Keys(SSE-S3): 기본설정으로 적용되는 Encryption 방법으로 S3에서 Key를 가진다.
    Encryption type은 AES-256으로 header에 "x-amz-server-side-encryption":"AES256"가 들어가 있어야 한다.
    새로 Bucket이나 Object 생성 시 기본적으로 적용된다.
  - Server-Side Encryption with KMS Keys stored in AWS KMS(SSE-KMS): aws KMS service를 통해 key를 관리해 S3에 API를 제공한다.
    header에 "x-amz-server-side-encryption":"aws:kms"가 들어가 있어야 한다.
    S3에 접근하여도 KMS에 Decrypt API를 사용해야 Object를 활용할 수 있기에 2중 보안이다. 하지만 KMS에서 처리는 하는 Throughput이 S3보다 낮아 전체작업 성능이 떨어질 수 있다.
  - Server-Side Encryption with Customer-Provided Keys(SSE-C): Client가 Key와 Object를 같이 S3에 넘겨서 처리한다.
    Key를 Header에 담아 Server로 보내기 때문에 HTTPS가 필수적 사항이다.
- Client-Side Encryption: Client가 Key를 활용해 Object를 Encrypt하고 S3로 보낸다.

### S3-Encryption in transit(SSL/TLS)

전송 과정에서 Encryption이 되는 것으로 SSL/TLS라 불린다. 이를 위해서는 HTTPS를 사용해야 한다.
SSE-C의 경우 HTTPS가 강제되며, 나머지는 사용 추천만 한다.
나머지에 대해서도 이를 강제하려면 Bucket Policy 상에서 aws:SecureTransport를 true로 설정하면 된다. 이게 설정되면 header에 encryption 항목이 필수적으로 들어가야만 API call이 받아드려진다.

### S3 CORS

CORS: Cross-Origin Resource Sharing으로 다른 Origin에서 Request가 왔을 때 Server에서 resource에 대한 접근을 통제하는 보안 기능이다.
Orign = schema(protocol) + host(domain) + port 이다. ex) https://www.example.com에서는 port 443을 HTTPS를 사용하는 domain www.example.com을 의미한다.
Req를 보내는 Server의 Access-Control-Allow-Origin에 해당 origin이 포함되어 있으면 resource에 대한 접근이 허락되며 res를 보낸다.
S3에서도 이와 같은 보안 기능이 존재하여 만약 S3 상에서 Static website를 hosting하고 있으면서 다른 S3에 대한 Resource request를 보낸다면 다른 S3에의 Access-Control-Allow-Origin에 static website를 hosting하는 S3의 origin을 추가시켜줘야 한다.

### S3 MFA Delete

Object에 Deletion 작업 수행하는 것에 대한 추가적인 보안 체계이다. 이를 사용하기 위해서는 Versioning 설정이 사전에 on되야 한다.
오직 root user만 이 설정을 사용할 수 있다.
MFA Delete는 object version을 영구적인 삭제를 수행하거나 Versioning을 off할 때 요구된다.

### S3 Access Logs

S3에서 발생한 모든 접근들을 다른 S3에서 log로 저장하는 기능이다. 이를 위해 Bucket들은 서로 같은 Region에 속해야 한다.
만약 Bucket이 자기 자신의 log를 저장하는 경우 log 생성이 또다른 log 생성을 촉진하는 무한 루프가 발생하기에 유의한다.

### Pre-Signed URLs

임시적으로 URL을 제공하여 user가 해당 URL의 object나 path에 대해서만 GET(download) 또는 PUT(upload)를 허용되는 기능이다.
이러한 URL은 S3 Console 또는 AWS CLI에서 생성 가능하다.

### Glacier Valut Lock

WORM(Wirte Once Read Many) Model로 Object를 삭제로부터 안전하게 지키고 싶을 때 사용하는 Policy이다.

### S3 Object Lock

Policy가 아닌 하나 또는 여러 Object에 대해서만 Lock을 하여 WORM Model을 적용하고 싶을 때 사용한다.
총 4가지 모드가 존재한다.

- Retention mode - Compliance: object version이 overwrite될 수 없으며 어떠한 유저든 절대로 삭제할 수 없다. Retention 또한 설정된 period동안 누구도 설정을 바꿀 수 없다.
- Retention mode - Goverance: Compliance와 동일하게 Object를 지키지만 root user와 같이 IAM persmission을 가진 User가 Retention 또는 삭제 작업을 할 수 있는 좀더 유연한 Mode이다.
- Retention period: object를 정해진 기간동안 지키며 연장될 수 있다.
- Legal Hold: 어떠한 Retention period에도 영향받지 않고 영구적으로 Object가 지켜진다. s3:PutObjectLegalHold IAM Permission을 가진 user를 통해 제거 될 수 있다.

### S3 Access Points

S3의 path에 접근할 수 있는 Access Point를 바깥에 따로 둬 Policy를 좀더 쉽게 관리할 수 있다.
Bucket Policy와 별도로 Access Policy가 존재하며 Permission이 있는 User나 Service만 해당 Access Point를 통해 S3에 접근할 수 있다.
각 Access Points는 DNS 이름과 Access Policy를 가진다.
만약 VPC 내에서만 Access Point를 접근하게 하고 싶으면 VPC Endpoint를 별도로 추가한다. 이러한 경우 총 3가지 Policy(VPC Endpoint, Access Point, S3 Bucket)를 통해 S3 접근을 통제할 수 있다.

### S3 Object Lambda

Lambda function을 통해 S3에서 Object를 가져오는 중간에 원하는 작업을 Object에 대하여 수행할 수 있다.  
원하는 수행 작업마다 Access Point와 Lambda Function을 두면 목적에 맞게 Access Point에 접근하여 목적에 맞는 작업이 수행된 S3 Object를 가져올 수 잇다.  
Ex)
만약 Object에 민감한 정보가 포함되어 있다면 해당 Access Point에서는 데이터를 가져올 때 이를 Redacting한 후 Object를 보내준다.  
또 다른 Access Point에서는 Object에 추가적인 정보를 더하는 Lambda fucntion을 둬 원하는 목적에 맞는 Access Point에 접근하여 알맞는 형태의 Object를 받을 수 있다.
