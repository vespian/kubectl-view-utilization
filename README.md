# `view-utilization` - kubectl plugin to view utilization
---
[![Build Status](https://travis-ci.org/etopeter/kubectl-view-utilization.svg?branch=master)](https://travis-ci.org/etopeter/kubectl-view-utilization) [![Test Coverage](https://api.codeclimate.com/v1/badges/88ad27e772eac5a4e19d/test_coverage)](https://codeclimate.com/github/etopeter/kubectl-view-utilization/test_coverage) [![license](https://img.shields.io/github/license/etopeter/kubectl-view-utilization.svg)](https://github.com/etopeter/kubectl-view-utilization/blob/master/LICENSE) [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/etopeter/kubectl-view-utilization/issues)

# <img src="static/view-utilization.png" alt="view-utilization" width=48>view-utilization
Kubectl plugin that shows cluster resource utilization. It is written in BASH/awk and uses kubectl tool to gather information.
You can use it to estimate cluster capacity and see at a glance overprovisioned resoures with this simple command **`kubectl view-utilization`**.

## Installation

### Install with krew (Recommended)
1. [Install krew](https://github.com/GoogleContainerTools/krew) plugin manager for kubectl.
2. Run `kubectl krew install view-utilization`.

#### Update with krew

Krew makes update process very simple. To update to latest version run `kubectl krew upgrade view-utilization`

### Install with Curl
For Kubernetes 1.12 or newer:
```shell
# Get latest tag
VIEW_UTILIZATION_PATH=/usr/local/bin
VIEW_UTILIZATION_TAG=$(curl -s https://api.github.com/repos/etopeter/kubectl-view-utilization/releases/latest | grep  "tag_name"| sed -E 's/.*"([^"]+)".*/\1/')

# Download and unpack plugin
curl -sL "https://github.com/etopeter/kubectl-view-utilization/releases/download/${VIEW_UTILIZATION_TAG}/kubectl-view-utilization-${VIEW_UTILIZATION_TAG}.tar.gz" |tar xzvf - -C $VIEW_UTILIZATION_PATH

# Rename file if you want to use kubectl view-utilization or leave it if you want to invoke it with kubectl view utilization (with space between). Underscore between words allows kubernetes plugin to have hyphen between words.
mv $VIEW_UTILIZATION_PATH/kubectl-view-utilization $VIEW_UTILIZATION_PATH/kubectl-view_utilization

# Change permission to allow execution
chmod +x $VIEW_UTILIZATION_PATH/kubectl-view_utilization

# Check if plugin is detected
kubectl plugin list
```

### Dependencies
While we try to be as minimalistic as possible the only dependency is AWK.

- kubectl
- bash
- awk (gawk, mawk, awk)

## Usage
This plugin should be invoked with kubectl command, and will appear as subcommand. It will use the existing context configured in `$KUBECONFIG` file. You can override context with `--context` parameter.

```shell
kubectl view-utilization                          
Resource     Requests  %Requests        Limits  %Limits   Allocatable   Schedulable         Free
CPU             43475         81         70731      132         53200          9725            0
Memory    94371840000         42  147184418816       66  222828834816  128456994816  75644416000
```

| Column      | Short | Description |
|-------------|-------|-------------|
| Requests    | Req   | Calculated total pod requests across all namespaces |
| %Requests   | %R    | Percentage of total requests agains allocatable requests |
| Limits      | Lim   | Calculated total pod limits across all namespaces  |
| %Limits     | %L    | Percentage of tatal limits agains allocatable limits |
| Allocatable | Alloc | Available allocatable resources |
| Schedulable | Sched | Resources that can be used to schedule pods; Available for pod requests (allocatable - requests) |
| Free        | Free  | Resources that are outside all requests or limits |


Example usage:

Human readable format `-h`
```shell
kubectl view-utilization -h
Resource  Req   %R   Lim    %L  Alloc  Sched  Free
CPU        43  71%    71  117%     60     17     0
Memory    88G  37%  138G   58%   237G   149G   99G
```
Check utilization for specific namespace `-n`

```shell
kubectl view-utilization -h -n kube-system
Resource   Req  %R   Lim  %L  Alloc  Sched  Free
CPU        3.7  6%   4.3  7%     60     57    56
Memory    5.4G  2%  7.9G  3%   237G   232G  229G
```

Check utilization for node groups using label filters.
Example filter results only for nodes in availability zone us-west-2b `failure-domain.beta.kubernetes.io/zone=us-west-2b`:

```shell
kubectl view-utilization -l failure-domain.beta.kubernetes.io/zone=us-west-2b -h
Resource  Req   %R  Lim    %L  Alloc  Sched  Free
CPU        14  64%   24  106%     22      8     0
Memory    30G  33%  47G   52%    89G    59G   42G
```

Overview of namespace utilization `kubectl view-utilization namespaces`
```shell
kubectl view-utilization namespaces -h
             CPU        Memory      
Namespace     Req  Lim   Req   Lim
analitics     6.6   10   14G   21G
kube-system   3.5  4.2  5.1G  7.6G
lt             13   21   27G   42G
monitoring   0.35  3.5  1.8G  3.5G
qa             13   21   27G   42G
rc            6.6   10   14G   21G
```

Output to JSON format.
```shell
kubectl view-utilization -o json | jq
{
  "CPU": {
    "requested": 43740,
    "limits": 71281,
    "allocatable": 60800,
    "schedulable": 17060,
    "free": 0
  },
  "Memory": {
    "requested": 94942265344,
    "limits": 148056834048,
    "allocatable": 254661525504,
    "schedulable": 159719260160,
    "free": 106604691456
  }
}
```

---

## Simplify workflow with aliases

Add to your `~/.bash_profile` or `~/.zshrc`

```shell
alias kvu="kubectl view-utilization -h"
```

Now you can use `kvu` alias to quickly show resource usage

Example commands:
```shell
kvu
kvu namespaces
kvu -n kube-system
```

---

## Change log

See the [CHANGELOG](CHANGELOG.md) file for details.


## Developing

1. Clone this repo with git
2. Test locally with kubectl pointing to your cluster (minikube or full cluster)
3. Run unit tests `make test`
