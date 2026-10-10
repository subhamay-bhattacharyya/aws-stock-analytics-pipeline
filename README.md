# Real-time stock market data analytics pipeline built on AWS

<!-- Row 1: Status - Most Important -->
[![Release](https://img.shields.io/github/v/release/subhamay-bhattacharyya/aws-stock-analytics-pipeline?label=Release)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/releases)&nbsp;[![Release Workflow](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/actions/workflows/release.yaml)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/badge/Languages-Python%20%7C%20YAML-blue)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya/aws-stock-analytics-pipeline)](https://github.com/subhamay-bhattacharyya/aws-stock-analytics-pipeline/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/07e4accbf35b23c2069188b1dd56e2b5/raw/aws-stock-analytics-pipeline.json?)](https://gist.github.com/bsubhamay/07e4accbf35b23c2069188b1dd56e2b5)

Real-time stock market data analytics pipeline on AWS. Market data is streamed through Kinesis, processed by Lambda, stored in S3 and DynamoDB, queried with Athena, and turned into trend alerts with SNS.

## Architecture

Green nodes are provisioned by [cloudformation/template.yaml](cloudformation/template.yaml). Grey dashed nodes are part of the target architecture and are not in the template yet.

```mermaid
flowchart LR
    User((User))
    PY[Python script<br/>yfinance fetcher]
    KDS[Kinesis Data Streams]
    L1[Lambda<br/>process, clean, transform]
    S3RAW[(S3<br/>raw stock data)]
    DDB[(DynamoDB<br/>processed data)]
    GLUE[Glue Data Catalog]
    ATH[Amazon Athena]
    S3RES[(S3<br/>query results)]
    L2[Lambda<br/>SMA-5 / SMA-20 signals]
    SNS[Amazon SNS]

    User --> PY
    PY -->|real-time stock data| KDS
    KDS -->|trigger| L1
    L1 -->|store raw data| S3RAW
    L1 -->|store processed data| DDB
    S3RAW -->|schema and data structured| GLUE
    GLUE -->|query| ATH
    ATH -->|store query results| S3RES
    DDB -->|trigger| L2
    L2 -->|notify trends| SNS
    SNS -.->|Email / SMS alerts| User

    classDef deployed fill:#d5f5e3,stroke:#1e8449,color:#000
    classDef planned fill:#f2f3f4,stroke:#7f8c8d,stroke-dasharray:5 5,color:#555
    class S3RAW,DDB,L1 deployed
    class PY,KDS,GLUE,ATH,S3RES,L2,SNS planned
```

## Deployed Resources

The root stack in [cloudformation/template.yaml](cloudformation/template.yaml) creates the following through nested stacks:

| Resource | Purpose |
| --- | --- |
| S3 bucket | Versioned storage, public access blocked, optional KMS encryption |
| DynamoDB table (`Receipts`) | Keyed on `symbol` and `timestamp`, with streams and point-in-time recovery |
| Lambda function | Python 3.12 processor, 256 MB, 60 s timeout |
| Lambda execution role | IAM role with S3, DynamoDB, Textract, SES, KMS, and CloudWatch Logs access |
| CloudWatch log group | Lambda logs, 14-day retention |
| S3 invoke permission | Lets the bucket invoke the Lambda function |

The Lambda function is currently triggered by S3 uploads. The Kinesis trigger shown in the diagram is planned.

## Deployment

The nested templates must be uploaded to the S3 bucket named by `NestedStacksS3BucketName` before deploying the root stack. Parameter defaults are in [cloudformation/parameters.json](cloudformation/parameters.json).

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name aws-stock-analytics-pipeline-stack \
  --capabilities CAPABILITY_NAMED_IAM
```

## License

MIT
