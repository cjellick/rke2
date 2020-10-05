# RKE2
![RKE2](docs/assets/logo-horizontal-rke.svg)

RKE2, also known as RKE Government, is Rancher's next-generation Kubernetes distribution.

It is a fully conformant ([currently under review](https://github.com/cncf/k8s-conformance/pull/1151)) Kubernetes distribution that focuses on security and compliance within the U.S. Federal Government sector.

To meet these goals, RKE2 does the following:

- Provides [defaults and configuration options](security/hardening_guide.md) that allow clusters to pass the [CIS Kubernetes Benchmark](security/cis_self_assessment.md) with minimal operator intervention
- Enables [FIPS 140-2 compliance](security/fips_support.md)
- Supports SELinux policy and [Multi-Category Security (MCS)](https://selinuxproject.org/page/NB_MLS) label enforcement
- Regularly scans components for CVEs using [trivy](https://github.com/aquasecurity/trivy) in our build pipeline

For more information and detailed installation and operation instructions, [please visit our docs](https://docs.rke2.io/).

## Quick Start
Here's the ***extremely*** quick start:
```sh
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
# Wait a bit
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
kubectl get nodes
```
For a bit more, [check out our full quick start guide](https://docs.rke2.io/install/quickstart/).

## Configuration File

The primary way to configure RKE2 is through its [config file](https://docs.rke2.io/install/install_options/install_options/#configuration-file). Command line arguments and environment variables are also available, but RKE2 is installed as a systemd service and thus these are not as easy to leverage.

By default, RKE2 will launch with the values present in the YAML file located at `/etc/rancher/rke2/config.yaml`.

An example of a basic `server` config file is below:

```yaml
# /etc/rancher/rke2/config.yaml
write-kubeconfig-mode: "0644"
tls-san:
  - "foo.local"
node-label:
  - "foo=bar"
  - "something=amazing"
```

In general, cli arguments map to their respective yaml key, with repeatable cli args being represented as yaml lists. So, an identical configuration using solely cli arguments is shown below to demonstrate this:

```bash
rke2 server \
  --write-kubeconfig-mode "0644"    \
  --tls-san "foo.local"             \
  --node-label "foo=bar"            \
  --node-label "something=amazing"
```

It is also possible to use both a configuration file and cli arguments.  In these situations, values will be loaded from both sources, but cli arguments will take precedence.  For repeatable arguments such as `--node-label`, the cli arguments will overwrite all values in the list.

Finally, the location of the config file can be changed either through the cli argument `--config FILE, -c FILE`, or the environment variable `$RKE2_CONFIG_FILE`.

## RPM Repositories

Signed RPMs are published for RKE2 within the `rpm-testing.rancher.io` and `rpm.rancher.io` RPM repositories. If you run the https://get.rke2.io script on nodes supporting RPMs, it will use these RPM rpeos by default. But you can also install them yourself.

The RPMs provide `systemd` units for managing `rke2`, but will need to be configured via configuration file before starting the services for the first time.

### Enterprise Linux 7

In order to use the RPM repository, on a CentOS 7 or RHEL 7 system, run the following bash snippet:

```bash
cat << EOF > /etc/yum.repos.d/rancher-rke2-1-18-latest.repo
[rancher-rke2-common-latest]
name=Rancher RKE2 Common Latest
baseurl=https://rpm.rancher.io/rke2/latest/common/centos/7/noarch
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key

[rancher-rke2-1-18-latest]
name=Rancher RKE2 1.18 Latest
baseurl=https://rpm.rancher.io/rke2/latest/1.18/centos/7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF
```

### Enterprise Linux 8

In order to use the RPM repository, on a CentOS 8 or RHEL 8 system, run the following bash snippet:

```bash
cat << EOF > /etc/yum.repos.d/rancher-rke2-1-18-latest.repo
[rancher-rke2-common-latest]
name=Rancher RKE2 Common Latest
baseurl=https://rpm.rancher.io/rke2/latest/common/centos/8/noarch
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key

[rancher-rke2-1-18-latest]
name=Rancher RKE2 1.18 Latest
baseurl=https://rpm.rancher.io/rke2/latest/1.18/centos/8/x86_64
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF
```

### Installing

After this, you can either run:

```bash
yum -y install rke2-server
```
or

```bash
yum -y install rke2-agent
```

The RPM will install a corresponding `rke2-server.service` or `rke2-agent.service` systemd unit that can be invoked like: `systemctl start rke2-server`. Make sure that you configure `rke2` before you start it, by following the `Configuration File` instructions below.

## FAQ

- [How is the different from RKE1 or K3s?](https://docs.rke2.io/#how-is-this-different-from-rke-or-k3s)
- [Why two names?](https://docs.rke2.io/#why-two-names)
