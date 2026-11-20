# OAC
**OAC(Origin Access Control)란?** S3 버킷은 비공개로 잠가두고 내 CloudFront 배포만 들어올 수 있게 하는 출입증이다. 

## 왜 필요할까?
S3를 public으로 열고 CloudFront를 앞에 붙이면 사용자가 CloudFront를 거치지 않고 S3 주소로 바로 들어올 수 있다.

```
사용자 → CloudFront → S3     ← 정상 경로
사용자 ──────────────→ S3     ← 우회 경로 (public이면 가능)
```

 우회하면 아래와 같은 문제가 생긴다.

- 캐시를 안 타서 S3 요청/전송 비용이 그대로 나감
- CloudFront에 걸어둔 WAF, HTTPS 강제, 국가 차단이 전부 무시됨

## 동작 원리
1. CloudFront가 S3에 요청할 때 **서명(SigV4)**을 붙여서 보낸다.
2. S3는 버킷 정책을 보고 이 서명이 내 CloudFront 배포가 맞는지 확인한다.
3. 맞으면 통과하고 아니면 403이 뜬다.

```json
// 버킷 정책 예시
{
  "Effect": "Allow",
  "Principal": { "Service": "cloudfront.amazonaws.com" },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EXXXXXXX"
    }
  }
}
```

| 부분 | 의미 |
| --- | --- |
| Principal | CloudFront 서비스에게 |
| Action | 파일 읽기만 허용 |
| Condition → AWS:SourceArn | 단, 이 배포일 때만 |

# OAI와 OAC
예전에는 **OAI(Origin Access Identity)**를 썼지만 지금은 OAC가 권장이다. 

|  | OAI(레거시) | OAC(권장) |
| --- | --- | --- |
| KMS 암호화 파일 | 못 읽음 | 읽을 수 있음 |
| 2022년 12월 이후 생긴 리전 | 지원 안 함 | 지원 |
| 쓰기 요청 (PUT, DELETE) | 제한적 | 지원 |

## Terraform 예시
```hcl
resource "aws_cloudfront_origin_access_control" "this" {
  name                              = "my-bucket-oac"
  origin_access_control_origin_type = "s3"
  signing_behavior                  = "always"   # 항상 서명
  signing_protocol                  = "sigv4"
}

resource "aws_cloudfront_distribution" "this" {
  origin {
    domain_name              = aws_s3_bucket.this.bucket_regional_domain_name  # REST 엔드포인트
    origin_id                = "s3-origin"
    origin_access_control_id = aws_cloudfront_origin_access_control.this.id
  }
  # ...
}
```

여기에 **버킷 정책(aws_s3_bucket_policy)**을 따로 붙여야 한다. OAC만 만들고 버킷 정책을 빼먹으면 전부 403이다.

# 주의할 점
- **S3 웹사이트 엔드포인트는 안 된다** : `bucket.s3-website...` 주소로 연결하면 OAC를 못 쓴다. REST 엔드포인트(`bucket_regional_domain_name`)를 써야 한다.
- **없는 파일은 404가 아니라 403** : 버킷 정책에 목록 조회(ListBucket) 권한이 없어서 S3가 파일이 없다는 사실조차 알려주지 않는다. 정상 동작이다.
- **KMS 암호화 버킷이면 키 정책도 필요** : KMS 키 정책에도 CloudFront의 복호화를 허용해야 한다.