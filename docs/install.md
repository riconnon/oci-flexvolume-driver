# Install

We publish the flexvolume driver as a single binary that needs to be installed on every node in your Kubernetes cluster.

## Ansible

The recommended way to install the driver is with ansible.

Compile the oci binary and ansible scripts

```
make build
```

Now you can use ansible to deploy the driver to your cluster

```
cd dist/ansible
```

Create an inventory file

```
cp hosts.example hosts
```

Add the details for all the masters and nodes in your cluster
(the oci-flexvolume-driver needs to be installed on all masters and workers)

```
[all:vars]
ansible_ssh_user=ubuntu

[kubemasters]
master ansible_ssh_host=...

[kubeslaves]
slave1 ansible_ssh_host=...
```

#### Run the playbook.

*If you're using Ubuntu 16.04 you might need to use python3 as in the example below.*

```
ansible-playbook -i hosts \
--private-key=generated/instances_id_rsa \
-e 'ansible_python_interpreter=/usr/bin/python3'
```

## Manually

The driver should be installed in the volume plugin path on **every**
node in your Kubernetes cluster at the following location:
`/usr/libexec/kubernetes/kubelet-plugins/volume/exec/oracle~oci/oci`.

NOTE: If running kube-controller-manager in a container you _must_ ensure that
the plugin directory is mounted into the container. See:
https://gitlab-odx.oracle.com/odx/oke-prime/merge_requests/15 for specifics.

### Configuration

The driver requires API credentials for a OCI account with the ability
to attach and detach [OCI block storage volumes][1] from to/from the appropriate
nodes in the cluster.

These credentials should be provided via a JSON file present on the nodes in the
cluster at `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/oracle~oci/flexvolume_driver.json`
in the following format:

```
{
    "user_ocid": "ocid1.user.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "compartment_ocid": "ocid1.compartment.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "region": "us-phoenix-1",
    "tenancy_ocid": "ocid1.tenancy.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "fingerprint": "d4:1d:8c:d9:8f:00:b2:04:e9:80:09:98:ec:f8:42:7e",
    "key_file": "/usr/libexec/kubernetes/kubelet-plugins/volume/exec/oracle~oci/flexvolume_driver.pem"
}
```

The signing key corresponding to the OCI user should be provided at the path
referenced by the `"key_file"` field (e.g.
`/usr/libexec/kubernetes/kubelet-plugins/volume/exec/oracle~oci/flexvolume_driver.pem`.)

If `"region"` and/or `"compartment_ocid"` are not specified in the config file
they will be retrieved from the hosts [OCI metadata service][1].

#### Extra configuration values

You can set these in the environment to override the default values.

* `OCI_FLEX_DRIVER_LOG_DIR` - Directory where the log file is written (Default:/usr/libexec/kubernetes/kubelet-plugins/volume/exec/oracle~oci)

[1]: https://docs.us-phoenix-1.oraclecloud.com/Content/Compute/Tasks/gettingmetadata.htm