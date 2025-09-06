# Static-Website
Project 3: AWS S3 + CloudFront Static Website Hosting
Goal: Host a static website using S3 and serve it globally via CloudFront.

Folder Structure
s3-static-website/
│── main.tf
│── variables.tf
│── outputs.tf
│── providers.tf
│── website/
│    └── index.html


provider.tf
provider "aws" {
  region  = "us-east-1"
  profile = "default"
}

variables.tf
variable "bucket_name" {
  default = "ravi-static-site-2025"

}

main.tf

resource "aws_s3_bucket" "static_site" {
  bucket = var.bucket_name
  acl    = "public-read"

  website {
    index_document = "index.html"
    error_document = "index.html"
  }
}

resource "aws_s3_bucket_policy" "public_access" {
  bucket = aws_s3_bucket.static_site.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = ["s3:GetObject"]
        Resource  = "${aws_s3_bucket.static_site.arn}/*"
      }
    ]
  })
}

resource "aws_s3_bucket_object" "index" {
  bucket       = aws_s3_bucket.static_site.bucket
  key          = "index.html"
  source       = "${path.module}/website/index.html"
  content_type = "text/html"
  acl          = "public-read"
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.static_site.bucket_regional_domain_name
    origin_id   = "s3-origin"
  }

  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "s3-origin"
    viewer_protocol_policy = "redirect-to-https"
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

output.tf
output "s3_bucket_name" {
  value = aws_s3_bucket.static_site.bucket
}

output "cloudfront_domain_name" {
  value = aws_cloudfront_distribution.cdn.domain_name
}


index.html
<!DOCTYPE html>
<html>
<head>
  <title>My Static Site</title>
</head>
<body>
  <h1>Hello from Ravi Kushwah!</h1>
</body>
</html>
