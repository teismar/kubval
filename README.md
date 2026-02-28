# kubval: K8s Manifest Validation Pipeline

A "Keep It Simple, Stupid" (KISS) approach to validating Kubernetes manifests. This repository implements a layered defense strategy to catch errors before they reach your cluster.

![Version](https://img.shields.io/badge/version-1.0.0-blue?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Security-Trivy](https://img.shields.io/badge/security-trivy-blueviolet?style=flat-square)
![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/teismar/kubval/k8s-validate.yaml?style=flat-square&label=pipeline)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=flat-square&logo=kubernetes&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Fedora-blue?style=flat-square&logo=fedora)

## The Validation Funnel

We chain four industry-standard tools to ensure your YAML is perfect:

    
- YAML Lint (`yamllint`): Structural integrity (indentation, tabs, and syntax).
- Schema Check (`kubeconform`): Kubernetes API compliance (data types and required fields).
- Security Scan (`trivy`): Security best practices (privileged containers and root users).
- Operational Quality (`kube-score`): Reliability checks (probes and resource limits).

## Local Usage (Fedora/Zsh)

1. Run the Pipeline

Run these commands in order to validate your `demo.yaml`:
Bash

### Step 1: Structural Check
`yamllint demo.yaml`

### Step 2: Schema Check
`kubeconform -summary -strict demo.yaml`

### Step 3: Security Scan
`trivy config demo.yaml`

### Step 4: Quality Score
`podman run --rm -v $(pwd):/project:Z zegl/kube-score:latest score /project/demo.yaml`

## GitHub Action Integration

Add this to `.github/workflows/k8s-validate.yaml` to automate your checks:

```yml
name: kubval
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: YAML Lint
        uses: ibiqlik/action-yamllint@v3
        continue-on-error: true

      - name: Kubeconform
        uses: docker://ghcr.io/yannh/kubeconform:latest
        continue-on-error: true
        with:
          args: "-summary -strict ."

      - name: Trivy Scan
        uses: aquasecurity/trivy-action@0.33.1
        continue-on-error: true
        with:
          scan-type: 'config'
          severity: 'HIGH,CRITICAL'

      - name: Kube-score
        run: docker run --rm -v $(pwd):/project:Z zegl/kube-score:latest score /project/*.yaml
```
