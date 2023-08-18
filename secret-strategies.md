There are many ways to manage secrets in a Kubernetes environment. Here are a few strategies:

Kubernetes Secrets: You're already using this approach. It allows you to store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys, and you can inject these secrets into pods as data volumes or environment variables. However, it has limitations. The secret data is still encoded using base64 and can be easily decoded. It also does not provide auditing or access control to the secrets.

Using an external secret management tool: Tools such as HashiCorp's Vault, AWS Secrets Manager, Azure Key Vault, or Google's Secret Manager offer a more robust and full-featured solution for managing secrets. These tools provide encryption in transit and at rest, detailed audit logs, and sophisticated access control mechanisms.

Kubernetes External Secrets: Kubernetes External Secrets allows you to securely and automatically inject secrets into your applications with the benefits of using a full-featured cloud-agnostic key management store like HashiCorp's Vault, AWS Secrets Manager, and others. It lets you retrieve secrets from external secret management systems and delivers them to your pods.

Using sealed secrets: Sealed Secrets is an open-source solution by Bitnami, which provides a controller and tool for one-way encrypted secrets. You can encrypt a secret object so that you can safely store it in a public repository or elsewhere.

Using Helm Secrets: Helm Secrets is a Helm plugin that allows you to encrypt, decrypt, and view secrets files and store them in secure storage.

CyberArk Conjur: CyberArk Conjur is an open-source security service that integrates with popular CI/CD tools to secure secrets, provide machine identity authorization, and more.

git-secret: git-secret is a bash tool to store your private data inside a git repo.

As an important note, it's essential to follow the principle of least privilege when dealing with secrets - only provide access to the secrets to the entities that absolutely need them. Also, avoid embedding secrets directly in your application code.