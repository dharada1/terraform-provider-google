---
# ----------------------------------------------------------------------------
#
#     ***     AUTO GENERATED CODE    ***    Type: MMv1     ***
#
# ----------------------------------------------------------------------------
#
#     This file is automatically generated by Magic Modules and manual
#     changes will be clobbered when the file is regenerated.
#
#     Please read more about how to change this file in
#     .github/CONTRIBUTING.md.
#
# ----------------------------------------------------------------------------
subcategory: "Compute Engine"
layout: "google"
page_title: "Google: google_compute_global_forwarding_rule"
sidebar_current: "docs-google-compute-global-forwarding-rule"
description: |-
  Represents a GlobalForwardingRule resource.
---

# google\_compute\_global\_forwarding\_rule

Represents a GlobalForwardingRule resource. Global forwarding rules are
used to forward traffic to the correct load balancer for HTTP load
balancing. Global forwarding rules can only be used for HTTP load
balancing.

For more information, see
https://cloud.google.com/compute/docs/load-balancing/http/



<div class = "oics-button" style="float: right; margin: 0 0 -15px">
  <a href="https://console.cloud.google.com/cloudshell/open?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fterraform-google-modules%2Fdocs-examples.git&cloudshell_working_dir=global_forwarding_rule_http&cloudshell_image=gcr.io%2Fgraphite-cloud-shell-images%2Fterraform%3Alatest&open_in_editor=main.tf&cloudshell_print=.%2Fmotd&cloudshell_tutorial=.%2Ftutorial.md" target="_blank">
    <img alt="Open in Cloud Shell" src="//gstatic.com/cloudssh/images/open-btn.svg" style="max-height: 44px; margin: 32px auto; max-width: 100%;">
  </a>
</div>
## Example Usage - Global Forwarding Rule Http


```hcl
resource "google_compute_global_forwarding_rule" "default" {
  name       = "global-rule"
  target     = google_compute_target_http_proxy.default.id
  port_range = "80"
}

resource "google_compute_target_http_proxy" "default" {
  name        = "target-proxy"
  description = "a description"
  url_map     = google_compute_url_map.default.id
}

resource "google_compute_url_map" "default" {
  name            = "url-map-target-proxy"
  description     = "a description"
  default_service = google_compute_backend_service.default.id

  host_rule {
    hosts        = ["mysite.com"]
    path_matcher = "allpaths"
  }

  path_matcher {
    name            = "allpaths"
    default_service = google_compute_backend_service.default.id

    path_rule {
      paths   = ["/*"]
      service = google_compute_backend_service.default.id
    }
  }
}

resource "google_compute_backend_service" "default" {
  name        = "backend"
  port_name   = "http"
  protocol    = "HTTP"
  timeout_sec = 10

  health_checks = [google_compute_http_health_check.default.id]
}

resource "google_compute_http_health_check" "default" {
  name               = "check-backend"
  request_path       = "/"
  check_interval_sec = 1
  timeout_sec        = 1
}
```
<div class = "oics-button" style="float: right; margin: 0 0 -15px">
  <a href="https://console.cloud.google.com/cloudshell/open?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fterraform-google-modules%2Fdocs-examples.git&cloudshell_working_dir=global_forwarding_rule_internal&cloudshell_image=gcr.io%2Fgraphite-cloud-shell-images%2Fterraform%3Alatest&open_in_editor=main.tf&cloudshell_print=.%2Fmotd&cloudshell_tutorial=.%2Ftutorial.md" target="_blank">
    <img alt="Open in Cloud Shell" src="//gstatic.com/cloudssh/images/open-btn.svg" style="max-height: 44px; margin: 32px auto; max-width: 100%;">
  </a>
</div>
## Example Usage - Global Forwarding Rule Internal


```hcl
resource "google_compute_global_forwarding_rule" "default" {
  provider              = google-beta
  name                  = "global-rule"
  target                = google_compute_target_http_proxy.default.id
  port_range            = "80"
  load_balancing_scheme = "INTERNAL_SELF_MANAGED"
  ip_address            = "0.0.0.0"
  metadata_filters {
    filter_match_criteria = "MATCH_ANY"
    filter_labels {
      name  = "PLANET"
      value = "MARS"
    }
  }
}

resource "google_compute_target_http_proxy" "default" {
  provider    = google-beta
  name        = "target-proxy"
  description = "a description"
  url_map     = google_compute_url_map.default.id
}

resource "google_compute_url_map" "default" {
  provider        = google-beta
  name            = "url-map-target-proxy"
  description     = "a description"
  default_service = google_compute_backend_service.default.id

  host_rule {
    hosts        = ["mysite.com"]
    path_matcher = "allpaths"
  }

  path_matcher {
    name            = "allpaths"
    default_service = google_compute_backend_service.default.id

    path_rule {
      paths   = ["/*"]
      service = google_compute_backend_service.default.id
    }
  }
}

resource "google_compute_backend_service" "default" {
  provider              = google-beta
  name                  = "backend"
  port_name             = "http"
  protocol              = "HTTP"
  timeout_sec           = 10
  load_balancing_scheme = "INTERNAL_SELF_MANAGED"

  backend {
    group                 = google_compute_instance_group_manager.igm.instance_group
    balancing_mode        = "RATE"
    capacity_scaler       = 0.4
    max_rate_per_instance = 50
  }

  health_checks = [google_compute_health_check.default.id]
}

data "google_compute_image" "debian_image" {
  provider = google-beta
  family   = "debian-9"
  project  = "debian-cloud"
}

resource "google_compute_instance_group_manager" "igm" {
  provider = google-beta
  name     = "igm-internal"
  version {
    instance_template = google_compute_instance_template.instance_template.id
    name              = "primary"
  }
  base_instance_name = "internal-glb"
  zone               = "us-central1-f"
  target_size        = 1
}

resource "google_compute_instance_template" "instance_template" {
  provider     = google-beta
  name         = "template-backend"
  machine_type = "e2-medium"

  network_interface {
    network = "default"
  }

  disk {
    source_image = data.google_compute_image.debian_image.self_link
    auto_delete  = true
    boot         = true
  }
}

resource "google_compute_health_check" "default" {
  provider           = google-beta
  name               = "check-backend"
  check_interval_sec = 1
  timeout_sec        = 1

  tcp_health_check {
    port = "80"
  }
}
```
## Example Usage - Private Service Connect Google Apis


```hcl
resource "google_compute_network" "network" {
  provider      = google-beta
  project       = "my-project-name"
  name          = "my-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "vpc_subnetwork" {
  provider                 = google-beta
  project                  = google_compute_network.network.project
  name                     = "my-subnetwork"
  ip_cidr_range            = "10.2.0.0/16"
  region                   = "us-central1"
  network                  = google_compute_network.network.id
  private_ip_google_access = true
}

resource "google_compute_global_address" "default" {
  provider      = google-beta
  project       = google_compute_network.network.project
  name          = "global-psconnect-ip"
  address_type  = "INTERNAL"
  purpose       = "PRIVATE_SERVICE_CONNECT"
  network       = google_compute_network.network.id
  address       = "100.100.100.106"
}

resource "google_compute_global_forwarding_rule" "default" {
  provider      = google-beta
  project       = google_compute_network.network.project
  name          = "globalrule"
  target        = "all-apis"
  network       = google_compute_network.network.id
  ip_address    = google_compute_global_address.default.id
  load_balancing_scheme = ""
}
```

## Argument Reference

The following arguments are supported:


* `name` -
  (Required)
  Name of the resource; provided by the client when the resource is
  created. The name must be 1-63 characters long, and comply with
  RFC1035. Specifically, the name must be 1-63 characters long and match
  the regular expression `[a-z]([-a-z0-9]*[a-z0-9])?` which means the
  first character must be a lowercase letter, and all following
  characters must be a dash, lowercase letter, or digit, except the last
  character, which cannot be a dash.

* `target` -
  (Required)
  The URL of the target resource to receive the matched traffic.
  The forwarded traffic must be of a type appropriate to the target object.
  For INTERNAL_SELF_MANAGED load balancing, only HTTP and HTTPS targets
  are valid.
  ([Beta](https://terraform.io/docs/providers/google/guides/provider_versions.html) only) For global address with a purpose of PRIVATE_SERVICE_CONNECT and
  addressType of INTERNAL, only "all-apis" and "vpc-sc" are valid.


- - -


* `description` -
  (Optional)
  An optional description of this resource. Provide this property when
  you create the resource.

* `ip_address` -
  (Optional)
  The IP address that this forwarding rule serves. When a client sends
  traffic to this IP address, the forwarding rule directs the traffic to
  the target that you specify in the forwarding rule. The
  loadBalancingScheme and the forwarding rule's target determine the
  type of IP address that you can use. For detailed information, refer
  to [IP address specifications](https://cloud.google.com/load-balancing/docs/forwarding-rule-concepts#ip_address_specifications).
  An address can be specified either by a literal IP address or a
  reference to an existing Address resource. If you don't specify a
  reserved IP address, an ephemeral IP address is assigned.
  The value must be set to 0.0.0.0 when the target is a targetGrpcProxy
  that has validateForProxyless field set to true.
  For Private Service Connect forwarding rules that forward traffic to
  Google APIs, IP address must be provided.

* `ip_protocol` -
  (Optional)
  The IP protocol to which this rule applies. When the load balancing scheme is
  INTERNAL_SELF_MANAGED, only TCP is valid. This field must not be set if the
  global address is configured as a purpose of PRIVATE_SERVICE_CONNECT
  and addressType of INTERNAL
  Possible values are `TCP`, `UDP`, `ESP`, `AH`, `SCTP`, and `ICMP`.

* `ip_version` -
  (Optional)
  The IP Version that will be used by this global forwarding rule.
  Possible values are `IPV4` and `IPV6`.

* `labels` -
  (Optional, [Beta](https://terraform.io/docs/providers/google/guides/provider_versions.html))
  Labels to apply to this forwarding rule.  A list of key->value pairs.

* `load_balancing_scheme` -
  (Optional)
  This signifies what the GlobalForwardingRule will be used for.
  The value of INTERNAL_SELF_MANAGED means that this will be used for
  Internal Global HTTP(S) LB. The value of EXTERNAL means that this
  will be used for External Global Load Balancing (HTTP(S) LB,
  External TCP/UDP LB, SSL Proxy)
  ([Beta](https://terraform.io/docs/providers/google/guides/provider_versions.html) only) Note: This field must be set "" if the global address is
  configured as a purpose of PRIVATE_SERVICE_CONNECT and addressType of INTERNAL.
  Default value is `EXTERNAL`.
  Possible values are `EXTERNAL` and `INTERNAL_SELF_MANAGED`.

* `metadata_filters` -
  (Optional)
  Opaque filter criteria used by Loadbalancer to restrict routing
  configuration to a limited set xDS compliant clients. In their xDS
  requests to Loadbalancer, xDS clients present node metadata. If a
  match takes place, the relevant routing configuration is made available
  to those proxies.
  For each metadataFilter in this list, if its filterMatchCriteria is set
  to MATCH_ANY, at least one of the filterLabels must match the
  corresponding label provided in the metadata. If its filterMatchCriteria
  is set to MATCH_ALL, then all of its filterLabels must match with
  corresponding labels in the provided metadata.
  metadataFilters specified here can be overridden by those specified in
  the UrlMap that this ForwardingRule references.
  metadataFilters only applies to Loadbalancers that have their
  loadBalancingScheme set to INTERNAL_SELF_MANAGED.
  Structure is [documented below](#nested_metadata_filters).

* `network` -
  (Optional, [Beta](https://terraform.io/docs/providers/google/guides/provider_versions.html))
  This field is not used for external load balancing.
  For INTERNAL_SELF_MANAGED load balancing, this field
  identifies the network that the load balanced IP should belong to
  for this global forwarding rule. If this field is not specified,
  the default network will be used.

* `port_range` -
  (Optional)
  This field is used along with the target field for TargetHttpProxy,
  TargetHttpsProxy, TargetSslProxy, TargetTcpProxy, TargetVpnGateway,
  TargetPool, TargetInstance.
  Applicable only when IPProtocol is TCP, UDP, or SCTP, only packets
  addressed to ports in the specified range will be forwarded to target.
  Forwarding rules with the same [IPAddress, IPProtocol] pair must have
  disjoint port ranges.
  Some types of forwarding target have constraints on the acceptable
  ports:
  * TargetHttpProxy: 80, 8080
  * TargetHttpsProxy: 443
  * TargetTcpProxy: 25, 43, 110, 143, 195, 443, 465, 587, 700, 993, 995,
                    1883, 5222
  * TargetSslProxy: 25, 43, 110, 143, 195, 443, 465, 587, 700, 993, 995,
                    1883, 5222
  * TargetVpnGateway: 500, 4500

* `project` - (Optional) The ID of the project in which the resource belongs.
    If it is not provided, the provider project is used.


<a name="nested_metadata_filters"></a>The `metadata_filters` block supports:

* `filter_match_criteria` -
  (Required)
  Specifies how individual filterLabel matches within the list of
  filterLabels contribute towards the overall metadataFilter match.
  MATCH_ANY - At least one of the filterLabels must have a matching
  label in the provided metadata.
  MATCH_ALL - All filterLabels must have matching labels in the
  provided metadata.
  Possible values are `MATCH_ANY` and `MATCH_ALL`.

* `filter_labels` -
  (Required)
  The list of label value pairs that must match labels in the
  provided metadata based on filterMatchCriteria
  This list must not be empty and can have at the most 64 entries.
  Structure is [documented below](#nested_filter_labels).


<a name="nested_filter_labels"></a>The `filter_labels` block supports:

* `name` -
  (Required)
  Name of the metadata label. The length must be between
  1 and 1024 characters, inclusive.

* `value` -
  (Required)
  The value that the label must match. The value has a maximum
  length of 1024 characters.

## Attributes Reference

In addition to the arguments listed above, the following computed attributes are exported:

* `id` - an identifier for the resource with format `projects/{{project}}/global/forwardingRules/{{name}}`

* `label_fingerprint` -
  ([Beta](https://terraform.io/docs/providers/google/guides/provider_versions.html))
  The fingerprint used for optimistic locking of this resource.  Used
  internally during updates.
* `self_link` - The URI of the created resource.


## Timeouts

This resource provides the following
[Timeouts](/docs/configuration/resources.html#timeouts) configuration options:

- `create` - Default is 4 minutes.
- `update` - Default is 4 minutes.
- `delete` - Default is 4 minutes.

## Import


GlobalForwardingRule can be imported using any of these accepted formats:

```
$ terraform import google_compute_global_forwarding_rule.default projects/{{project}}/global/forwardingRules/{{name}}
$ terraform import google_compute_global_forwarding_rule.default {{project}}/{{name}}
$ terraform import google_compute_global_forwarding_rule.default {{name}}
```

## User Project Overrides

This resource supports [User Project Overrides](https://www.terraform.io/docs/providers/google/guides/provider_reference.html#user_project_override).
