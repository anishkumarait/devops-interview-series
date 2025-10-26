**1. How do you prevent accidental resource deletion?**
To protect against accidental deleteion of critical resources, Terraform provides the `prevent_destroy` lifecycle argument. When this setting is enabled, any attempt to run `terraform destroy` or `apply` a configuration that removes or replaces the resource will fail. For example,
```
resource "aws_s3_bucket" "prod_data_bucket" {
  bucket = "my-production-data-bucket"

  lifecycle {
    prevent_destroy = true
  }
}
```
In this example, if you run `terraform destroy` or removes the S3 bucket definition from the Terraform code. Terraform will refuse to delete the bucket and show an error message such as *Resource `aws_s3_bucket.prod_data_bucket` has `lifecycle.prevent_destroy` set, but the plan calls for this resource to be destroyed.* 

---

**2. How do you handle provider API rate limiting?**
- **Implement exponential backoff:** Many providers (like AWS) include built-in retry mechanisms for throttled API calls. But you can also configure or tune these settings in the provider block using `max_retries` attribute for AWS.
- **Use `time_sleep` resource between operations:** This resource in Terraform can introduce deliberate pauses between resource creations or modifications. This is useful for preventing bursts of API requests that could exceed rate limits.
- **Split large deployments into smaller batches:** You can split resources into smaller and more manageable batches that helps minimize the number of concurrent API calls. You can achieve this by using `Terraform modules`, `workspaces` or the `-target` option to apply only a subset of resources at a time.

---

**3. How do you recover from courrupted state file?**
- **Recover from backup:** The quickest way to recover from courrupted state file is from backup, if backup is configured. You can restore the state by replacing the corrupted state file with the backup file. This is why it is always recommended to use remote state storage with versioning (such as S3 with versioning enabled) or Terraform Cloud/Enterprise.
- **`terraform import`:** If the backup is unavailable or incomplete, you can use `terraform import` to reconstruct the state for existing resources. This command allows Terraform to associate existing infrastructure with resource definitions in your configuration, effectively rebuilding the state. For example: `terraform import aws_s3_bucket.example_bucket my-bucket-name`

---

**4. How do you migrate state from one backend to another?**
Migration steps:
- Run `terraform state pull` to save the current state locally as a backup.
- Update your Terraform configuration to define the new backend (for example, S3, Terraform Cloud, or other supported backends).
- Run `terraform init -migrate-state` to initialize Terraform with the new backend and migrate the existing state automatically.
- Verify that the state has been successfully migrated by checking the new backend for the state file.
**Note**: Always perform backend migration during maintenance windows and ensure you have proper backups to avoid accidental data loss or service disruption.

---

**5. How do you handle state drift in production environments?**
State drift occurs when the real-world infrastructure differs from the Terraform state that can lead to inconsistencies and deployment issues. To handle drift in production, regularly run `terraform plan` in CI/CD pipelines to detect any differences between the actual infrastructure and the Terraform state.
If drift is detected, you can use `terraform import` to reconcile resources that exist in the environment but are missing from the state. This brings Terraform back in sync without recreating resources. 
Additionally, it iss a best practice to set up automated drift detection with monitoring and alerts so that your team is immediately notified of any manual or unexpected modifications in production infrastructure.

---

**6. How do you manage secrets securely in Terraform?**
- store secrets in external systems: Use tools such as  HashiCorp Vault, AWS Secrets Manager or Azure Key Vault to store sensitive values instead of hardcoding them in Terraform code.
- Mark variables as sensitive: Use the `sensitive = true` attribute in variable definitions to prevent values from being displayed in logs, plan output.
    ```
    variable "db_password" {
    type      = string
    sensitive = true
    }
    ```
- Encrypt remote state: Ensure that your remote state backend such as S3 or Terraform Cloud is encrypted at rest so that sensitive information stored in the state file is protected.

---

**7. How do you properly structure Terraform modules for enterprise use?**
- **Create reusable modules:** Each module should encapsulate a specific piece of infrastructure such as a VPC, S3 bucket or RDS instance. The module should be designed to work independently of other modules. Expose only the necessary input variables and output values, keeping the internals hidden for simplicity and reusability. Also, use semantic versioning for modules so different teams can pin specific versions safely.
- **Define clear interfaces:** Module should use descriptive input variables with default values where appropriate. ALso, it should expose key attributes required by other modules or stacks as outputs.
- **Dependency Management:** Modules should consume outputs from other modules or use remote state data sources to obtain information about existing resources. 
Example module structure:
```
terraform-repo/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── s3_bucket/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
└── README.md
```

---

**8. What happens if you change the `terraform.backend` configuration after a state file is already created?**
If you change the `terraform.backend` configuration after a state file is already created, Terraform will treat it as a change in the backend configuration. This can result in Terraform trying to migrate the state file from the old backend to the new one.

---

**9. What is the difference between `for_each` and `for` in Terraform?**
- **`for_each`:** It is used at the resource or module level to create multiple instances of a resource or module based on a map or set of strings. Each instance is identified by a unique key that makes it easier to manage, reference, and update individual resources. For example, if you want to create three S3 buckets dynamically, you can use `for_each`:
    ```
    variable "buckets" {
    type    = set(string)
    default = ["dev-bucket", "staging-bucket", "prod-bucket"]
    }

    resource "aws_s3_bucket" "example" {
    for_each = var.buckets
    bucket   = each.value
    }
    ```
- **`for`:** It is used inside expressions to generate lists or maps dynamically within a single resource, variable or output. It allows you to transform existing collections or create computed values without creating multiple resource instances. For example, you can use `for` to create a list of uppercase bucket names, but it does not create separate resources:
    ```
    variable "buckets" {
    type    = list(string)
    default = ["dev", "staging", "prod"]
    }

    output "uppercase_buckets" {
    value = [for b in var.buckets : upper(b)]
    }
    ```
**Note:** `for_each` controls resource/module creation while `for` is used for data transformation within expressions.

**10. What happens if a resource fails halfway through a `terraform apply`?**
If a resource fails halfway through a `terraform apply`, Terraform leaves the successfully created resources running but marks the failed resource as tainted in the state file. This means that the next time you run `terraform apply`, Terraform will attempt to recreate only the failed or tainted resources, This leaves you in a partial state until all resources are successfully applied.

