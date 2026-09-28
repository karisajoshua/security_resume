# AWS Resume API Challenge — design notes

This repository currently contains documentation for a proposed serverless resume API. It does **not** contain the Lambda source, Terraform files, Dockerfile, Kubernetes manifests, GitHub Actions workflow, or test evidence described in the earlier draft README. Treat the architecture and snippets below as a learning plan, not as a deployed or verified system.

## Proposed architecture

- API Gateway receives a request for resume data.
- AWS Lambda reads a record from DynamoDB and returns JSON.
- IAM limits the Lambda function to the table and actions it needs.
- Infrastructure configuration and a deployment workflow would make the build reproducible.

## Implementation checklist

1. Add a working Lambda handler and sample data with no personal information.
2. Add scoped Terraform configuration, including the table, role, function, API, and permissions.
3. Add automated tests for valid requests, missing records, and error handling.
4. Document local setup and deployment commands that have been run successfully.
5. Add a CI workflow and report its actual status.
6. Only then document container or Kubernetes deployment if implemented and necessary.

No live endpoint or deployment is claimed here. The repository will be updated as implementation and verification are completed.
