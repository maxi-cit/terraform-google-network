# Private Service Networks

This module enables the usage of [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect) for a specific subnetwork.

the resources created/managed by this module are:

- one private DNS zone to configure `private.googleapis.com.`
- one private DNS zone to configure `gcr.io.`
- one private DNS zone to configure `pdk.dev.`
- one Global Address resource to configure `Private Service Connect` endpoint
- one Global Forwarding Rule resource to forward traffic to respective HTTP(S) load balancing

# Usage

Basic usage of this module is as follows:

```hcl
module "private_service_connect" {
  source                     = "terraform-google-modules/network/google//modules/private_service_connect"

  project_id                 = "project-1234"
  network_self_link          = "<NETWORK SELF LINK>"
  private_service_connect_ip = "10.3.0.5"
  forwarding_rule_target     = "all-apis|vpc-sc"
}
```

Private Service Connect IP must fulfill requirements detailed [here](https://cloud.google.com/vpc/docs/configure-private-service-connect-apis#ip-address-requirements).

Target subnetwork must have Private Google Access enabled.

In case all Egress is restricted, you must configure a proper firewall rule. Following is an example.

```hcl
resource "google_compute_firewall" "allow_private_api_egress" {
  name      = "allow-google-apis-all-tcp-443"
  network   = "<network-name>"
  project   = "<project-id>"
  direction = "EGRESS"
  priority  = 65534 # this must be set accordingly

  dynamic "log_config" {
    for_each = var.firewall_enable_logging == true ? [{
      metadata = "INCLUDE_ALL_METADATA"
    }] : []

    content {
      metadata = log_config.value.metadata
    }
  }

  allow {
    protocol = "tcp"
    ports    = ["443"]
  }

  destination_ranges = [<PRIVATE SERVICE CONNECT IP>] # Output from Private Service Connect module

  target_tags = ["allow-google-apis"]
}
```


<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| dns\_code | Code to identify DNS resources in the form of `dz-{dns_code}-{dns_type}` | `string` | `""` | no |
| forwarding\_rule\_name | Forwarding rule resource name. The forwarding rule name for PSC Google APIs must be an 1-20 characters string with lowercase letters and numbers and must start with a letter. Defaults to `globalrule` | `string` | `"globalrule"` | no |
| forwarding\_rule\_target | Target resource to receive the matched traffic. Only `all-apis` and `vpc-sc` are valid. | `string` | n/a | yes |
| network\_self\_link | Network self link for Private Service Connect. | `string` | n/a | yes |
| private\_service\_connect\_ip | The internal IP to be used for the private service connect. | `string` | n/a | yes |
| private\_service\_connect\_name | Private Service Connect endpoint name. Defaults to `global-psconnect-ip` | `string` | `"global-psconnect-ip"` | no |
| project\_id | Project ID for Private Service Connect. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| dns\_zone\_gcr\_name | Name for Managed DNS zone for GCR |
| dns\_zone\_googleapis\_name | Name for Managed DNS zone for GoogleAPIs |
| dns\_zone\_pkg\_dev\_name | Name for Managed DNS zone for PKG\_DEV |
| forwarding\_rule\_name | Forwarding rule resource name. |
| forwarding\_rule\_target | Target resource to receive the matched traffic. Only `all-apis` and `vpc-sc` are valid. |
| global\_address\_id | An identifier for the global address created for the private service connect with format `projects/{{project}}/global/addresses/{{name}}` |
| private\_service\_connect\_ip | Private service connect ip |
| private\_service\_connect\_name | Private service connect name |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->