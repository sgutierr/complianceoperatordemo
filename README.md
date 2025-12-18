# Compliance Operator Demo

This repository contains examples and configurations for the OpenShift Compliance Operator, including installation via ArgoCD and multiple demos with custom CustomRules for different security and compliance scenarios.

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Examples and Demos](#examples-and-demos)
- [References](#references)

## Overview

The OpenShift Compliance Operator enables automated compliance scanning and remediation of security issues in Kubernetes/OpenShift clusters. This repository provides:

- **Automated installation** via ArgoCD
- **Practical examples** of CustomRules for different security scenarios
- **Ready-to-use configurations** for TailoredProfiles and ScanSettingBindings
- **Progressive demos** from basic concepts to complete CIS scans

## Repository Structure

```
compliance-operator/
├── examples/              # CustomRules examples and configurations
│   ├── demo1/            # Basic Security Checks
│   ├── demo2/            # NetworkPolicy Security Checks
│   ├── demo3/            # Container and Pod Security Best Practices
│   └── demo4/            # CIS Benchmark Compliance Scan
└── installation/         # Installation scripts and configurations
    ├── argocd/           # ArgoCD applications for automated installation
    └── operator/         # Manual operator installation
```

## Installation

### Option 1: Installation via ArgoCD (Recommended)

ArgoCD installation allows managing the Compliance Operator and its examples in a declarative and automated way.

#### Prerequisites

- ArgoCD installed in the cluster
- Permissions to create resources in the `openshift-compliance` namespace
- Access to the Git repository where this code is located

#### Installation Steps

1. **Configure RBAC permissions** (IMPORTANT - must be applied first):

```bash
oc apply -f compliance-operator/installation/argocd/argocd-rbac.yaml
```

2. **Apply ArgoCD applications**:

```bash
# Apply operator installation application
oc apply -f compliance-operator/installation/argocd/compliance-operator-installation.yaml

# Apply examples application
oc apply -f compliance-operator/installation/argocd/compliance-operator-examples.yaml
```

3. **Verify the installation**:

```bash
# Verify that the operator is installed
oc get csv -n openshift-compliance

# Verify operator pods
oc get pods -n openshift-compliance

# Verify ArgoCD applications
oc get applications -n argocd
```

For more details, see the [ArgoCD README](compliance-operator/installation/argocd/Readme.md).

### Option 2: Manual Installation

To install the operator manually:

```bash
# Create namespace
oc apply -f compliance-operator/installation/operator/namespace.yaml

# Create subscription
oc apply -f compliance-operator/installation/operator/susbcription.yaml
```

### Client Configuration (oc-compliance plugin)

To use the Compliance Operator utilities from your client machine, you need to install the `oc-compliance` plugin:

```bash
# Install podman (if not available)
sudo yum -y install podman

# Configure authentication with Red Hat registry
export REGISTRY_AUTH_PATH=~
mkdir -p $REGISTRY_AUTH_PATH/containers
oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json

# Download and install the plugin
export OC_PATH=$(whereis -b oc | awk '{ print $2 }' | sed 's/\/oc//g')
podman run --authfile $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json --rm --entrypoint /bin/cat registry.redhat.io/compliance/oc-compliance-rhel8 /usr/bin/oc-compliance > /tmp/oc-compliance && chmod +x /tmp/oc-compliance
mv /tmp/oc-compliance $OC_PATH/oc-compliance

# Install additional tools for reports
sudo yum -y install openscap-scanner tree bzip2

# Verify installation
oc compliance --help
```

For more details, see the [Installation README](compliance-operator/installation/Readme.md).

## Examples and Demos

### Demo 1: Basic Security Checks

This demo includes basic security checks for cluster administration, registry configuration, and detection of unapproved databases.

**Included CustomRules:**
- `cluster-admin-allow-list.yaml`: Audits cluster-admin access against an allow-list (aligned with CIS 5.1.1)
- `allowed-registries-configured.yaml`: Verifies that trusted registries are defined in the cluster image configuration
- `disallow-shadow-databases.yaml`: Prevents application teams from deploying unapproved database instances

**Apply the demo:**

```bash
oc apply -f compliance-operator/examples/demo1/
oc apply -f compliance-operator/examples/demo1/custom-security-scan.yaml
```

### Demo 2: NetworkPolicy Security Checks

This demo focuses on network security through NetworkPolicies.

**Included CustomRules:**
- `netpol-disallow-allow-all-in-labeled-namespaces.yaml`: Detects NetworkPolicies that allow all traffic in labeled namespaces
- `netpol-require-deny-all-in-labeled-namespaces.yaml`: Requires a NetworkPolicy that denies all traffic in labeled namespaces

**Apply the demo:**

```bash
oc apply -f compliance-operator/examples/demo2/
oc apply -f compliance-operator/examples/demo2/ScanSettingBinding.yaml
```

### Demo 3: Container and Pod Security Best Practices

This demo contains additional CustomRules examples covering security best practices for containers and pods.

**Included CustomRules:**
- `no-latest-image-tag.yaml`: Verifies that pods do not use the 'latest' tag for images
- `pods-must-have-resource-limits.yaml`: Ensures all containers have defined resource limits
- `pods-must-not-run-as-root.yaml`: Verifies that containers do not run as root (UID 0)
- `sec-must-not-be-in-env-vars.yaml`: Ensures sensitive data from Secrets is not exposed as environment variables
- `critical-namespaces-must-have-networkpolicy.yaml`: Verifies that critical namespaces have NetworkPolicy defined
- `pods-must-have-readiness-probe.yaml`: Ensures all containers have readiness probes configured
- `pods-must-not-use-hostnetwork.yaml`: Verifies that pods do not use hostNetwork
- `pods-must-not-use-privileged.yaml`: Ensures containers do not run in privileged mode
- `namespaces-must-have-pod-security-standards.yaml`: Verifies that namespaces have Pod Security Standards labels

**Apply the demo:**

```bash
# Apply all CustomRules
oc apply -f compliance-operator/examples/demo3/

# Apply the TailoredProfile and ScanSettingBinding
oc apply -f compliance-operator/examples/demo3/tailoredprofile-demo3.yaml
oc apply -f compliance-operator/examples/demo3/scansettingbinding-demo3.yaml

# Verify scan status
oc get compliancescans -n openshift-compliance
oc get compliancecheckresults -n openshift-compliance
```

For more details, see the [Demo 3 README](compliance-operator/examples/demo3/Readme.md).

### Demo 4: CIS Benchmark Compliance Scan

This demo shows how to run a compliance scan against the **CIS Red Hat OpenShift Container Platform 4 Benchmark**.

**Prerequisites:**
- OpenShift cluster
- Client host configured with oc-compliance plugin
- EBS as storage backend to support a 1GB persistent volume

**Apply the demo:**

```bash
# Create the ScanSettingBinding
oc compliance bind -N ocp4-cis-binding profile/ocp4-cis profile/ocp4-cis-node -n openshift-compliance

# Monitor the scan
oc get compliancescan -w -n openshift-compliance

# View results
oc get ComplianceCheckResult -n openshift-compliance

# View remediations
oc get ComplianceRemediation -n openshift-compliance

# Download raw results
mkdir -p ~/compliance-results
oc compliance fetch-raw scansettingbindings ocp4-cis-binding -o ~/compliance-results/ -n openshift-compliance
```

For more details, see the [Demo 4 README](compliance-operator/examples/demo4/Readme.md).

## Verification and Monitoring

### Verify scan status

```bash
# View running scans
oc get compliancescans -n openshift-compliance

# View verification results
oc get compliancecheckresults -n openshift-compliance

# View available remediations
oc get complianceremediations -n openshift-compliance
```

### View details of a specific result

```bash
oc compliance view-result <result-name> -n openshift-compliance
```

### Download raw results

```bash
oc compliance fetch-raw scansettingbindings <binding-name> -o <output-directory> -n openshift-compliance
```

## References

- [OpenShift Compliance Operator Documentation](https://docs.openshift.com/container-platform/latest/security/compliance_operator/compliance-operator-understanding.html)
- [Compliance Operator Customization Guide](https://ralvares.github.io/openshift-security-framework/docs/html/compliance-operator-customization.html)
- [Common Expression Language (CEL)](https://github.com/google/cel-spec)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [OpenSCAP Documentation](https://www.open-scap.org/)

## Contributions

This repository contains examples and configurations for the Compliance Operator. Feel free to adapt these examples to your specific needs.

## License

This repository contains reference examples and configurations for the OpenShift Compliance Operator.
