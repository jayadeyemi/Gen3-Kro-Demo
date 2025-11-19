## Related Project

The primary reference implementation for this work is:
- [indiana-university/gen3-kro](https://github.com/indiana-university/gen3-kro)

That repository demonstrates an opinionated pattern for provisioning and managing cloud infrastructure from Kubernetes, using Argo CD, Terraform, and cloud controllers.

### High-level flow

1. A VPC is deployed from a central `csoc` cluster into a target AWS account.
2. Terraform prepares the AWS account (IAM roles, policies, and annotations).
3. Argo CD, running in the `csoc` cluster, reconciles Kubernetes resources (CRDs, RGDs, and application manifests) from Git.
4. Cloud controllers (ACK/ASO/GCC) watch those resources and create/update the corresponding cloud infrastructure.
5. Developers work in their own repositories and namespaces, using templates to build applications and infrastructure safely.

### Terraform responsibilities

Terraform is responsible for preparing the AWS account and wiring it to the cluster:

- Creates IAM roles and policies required by the controllers and applications.
- Uses the IAM roles and policy definitions from the reference implementation.
- Injects the resulting IAM role ARNs into the cluster (for example, as annotations or secrets).
- Pre-annotates Kubernetes namespaces so Argo CD and the controllers can scope access and behavior.

### Argo CD responsibilities

Argo CD runs in the `csoc` cluster and drives the desired state:

- Installs required CRDs for:
  - Cloud controllers (ACK/ASO/GCC).
  - Cloud infrastructure resource graph definitions (RGDs).
- Deploys cloud controllers (ACK/ASO/GCC) that translate Kubernetes resources into cloud resources.
- Deploys applications into specific namespaces that have been pre-annotated by Terraform.
- Synchronizes `Kind` objects from spoke/customer Git repositories, using the graphs available in the `csoc` cluster to create the corresponding infrastructure components.

### Developer experience

This pattern is designed to let developers build cloud applications quickly and safely:

- Developers start from predefined templates (applications + infrastructure) stored in Git.
- Namespaced access and annotations control what each team or developer can manage.
- When developers push changes to their repositories:
  - Argo CD syncs those changes into the cluster.
  - Cloud controllers reconcile the infrastructure state in the target AWS account.
- This allows developers to:
  - Provision and manage infrastructure in their own AWS accounts.
  - Operate within the guardrails and access restrictions defined by platform/infra teams.
