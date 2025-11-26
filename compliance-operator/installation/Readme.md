We need to pull the oc-compliance binary plugin from the Red Hat registry and place it with the oc binary file on the client host.
https://docs.openshift.com/container-platform/4.8/security/compliance_operator/oc-compliance-plug-in-using.html
If podman is not available on the client host, we need to install it as it will be used to pull the container image that contains the oc-compliance plugin:
$ sudo yum -y install podman


To be able to access the Red Hat registry, podman needs an auth file with the credentials to authenticate to registry.redhat.io. We can download the pull secret from the Red Hat cloud console (https://console.redhat.com/openshift) or obtain the credentials from the OpenShift cluster. In this case, we will obtain it from the OpenShift cluster:

``` $ export REGISTRY_AUTH_PATH=~ && mkdir -p $REGISTRY_AUTH_PATH/containers && oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json ```


Once podman is configured to access the Red Hat registry, we just need to pull the plugin, install it within a location on the machine PATH and give execution permissions:

``` $ export OC_PATH=$(whereis -b oc | awk '{ print $2 }' | sed 's/\/oc//g') ```


``` $ podman run --authfile $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json --rm --entrypoint /bin/cat registry.redhat.io/compliance/oc-compliance-rhel8 /usr/bin/oc-compliance > /tmp/oc-compliance && chmod +x $/tmp/oc-compliance ```


``` $ mv /tmp/oc-compliance $OC_PATH/oc-compliance ``` 


We can verify the successful enablement of th plugin by running the appropiate oc target:

``` $ oc compliance --help ```

A set of utilities that come along with the compliance-operator.

Usage:
  oc-compliance [flags]
  oc-compliance [command]

Available Commands:
  bind        Creates a ScanSettingBinding for the given parameters
  controls    Get a report of what controls you're complying with
  fetch-fixes Download the fixes/remediations
  fetch-raw   Download raw compliance results
  help        Help about any command
  rerun-now   Force a re-scan for one or more ComplianceScans
  view-result View a ComplianceCheckResult

Flags:
  -h, --help   help for oc-compliance

Use "oc-compliance [command] --help" for more information about a command.


Now, we just need to install the remaining tools that will be used to generate the reports: the openscap scanner (https://www.open-scap.org/tools/openscap-base/#download), bzip2 and tree.

``` $ sudo yum -y install openscap-scanner tree bzip2 ```

Once all of the packages have been installed, the client host is ready to scan our cluster and generate compliance reports.
