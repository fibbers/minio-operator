# MinIO Operator Reference

[![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/k8s-operator.svg?maxAge=604800)](https://hub.docker.com/r/minio/k8s-operator)

This document explains the various fields supported in `MinIOInstance` yaml file and how to use these fields to deploy and access MinIO server clusters.

MinIO Operator creates native Kubernetes resources within the cluster. If the MinIOInstance is named as `minioinstance`, resources and their names as created by MinIO Operator are:

- Headless Service: `minioinstance-hl-svc`
- StatefulSet: `minioinstance`
- Secret: `minioinstance-tls` (If `requestAutoCert` is enabled)
- CertificateSigningRequest: `minioinstance-csr` (If `requestAutoCert` is enabled)

## MinIOInstance Fields

- `kind`: This defines the resource type to be created. MinIO Operator CRD defines the `kind` for MinIO server as `MinIOInstance`.

- `metadata`: This field allows a way to assign metadata to a MinIOInstance. Internally `metadata` is a struct type as [explained here](https://godoc.org/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta).

- `scheduler`: Set custom scheduler for pods created by MinIO Operator.

- `spec.metadata`: Define the object metadata to be passed to all the members pods of this MinIOInstance. This allows adding annotations and labels. For example,you can add Prometheus annotations here. Internally `metadata` is a struct type as [explained here](https://godoc.org/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta).

- `spec.image`: Set the container registry and image tag for MinIO server to be used in the MinIOInstance.

- `spec.zones`: Set the number of servers per MinIO Zone. Add a new Zone field to expand the MinIO cluster. Read more on [MinIO zones here](https://github.com/minio/minio/blob/master/docs/distributed/DESIGN.md).

- `spec.volumesPerServer`: Set the number of volume mounts per MinIO node. For example if you set `spec.zones[0].Servers = 4`, `spec.zones[1].Servers = 8` and `spec.volumesPerServer = 4`, then you'll have total 12 MinIO Pods, with 4 volume mounts on each Pod. Note that `volumesPerServer` is static per cluster, expanding a cluster will add new nodes.

- `spec.imagePullSecret`: Defines the secret to be used for pull image from a private Docker image.

- `spec.credsSecret`: Use this secret to assign custom credentials (access key and secret key) to MinIOInstance.

- `spec.replicas`: Define the number of nodes to be created for current MinIOInstance cluster.

- `spec.podManagementPolicy`: Define Pod Management policy for pods created by StatefulSet. This is set to `Parallel` by default. Refer [the documentation](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#pod-management-policy) for details.

- `spec.mountPath`: Set custom mount path. This is the path where PV gets mounted on MinIOInstance pods. This is set to `/export` by default.

- `spec.subPath`: Set custom sub-path under mount path. This is the directory under mount path where PV gets mounted on MinIOInstance pods. This is set to `""` by default.

- `spec.volumeClaimTemplate`: Specify the template to create Persistent Volume Claims for MinIOInstance pods.

- `spec.env`: Add MinIO specific environment variables to enable certain features.

- `spec.requestAutoCert`: Enable this to create use your Kubernetes cluster's root Certificate Authority (CA).

- `spec.certConfig`: When `spec.requestAutoCert` is enabled, use this field to pass additional parameters for certificate creation.

- `spec.externalCertSecret`: Set an external secret with private key and certificate to be used to enabled TLS on MinIOInstance pods. Note that only one of `spec.requestAutoCert` or `spec.externalCertSecret` should be enabled at a time. Follow [the document here](https://github.com/minio/minio/tree/master/docs/tls/kubernetes#2-create-kubernetes-secret) to create the secret to be passed in this section.

- `spec.resources`: Specify CPU and Memory resources for each MinIOInstance container. Refer [this document](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-types) for details.

- `spec.liveness`: Add liveness check for MinIOInstance containers. Refer [this document](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-a-liveness-command) for details.

- `spec.readiness`: Add readiness check for MinIOInstance containers. Only single node MinIOInstance container should enable readiness checks. Refer [this document](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-a-liveness-command) for details.

- `spec.nodeSelector`: Add a selector which must be true for the MinIOInstance pod to fit on a node. Refer [this document](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) for details.

- `spec.tolerations`: Define a toleration for the MinIOInstance pod to match on a taint. Refer [this document](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) for details.

`spec.securityContext`: Define a security context for the MinIOInstance pod. Refer [this document](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) for details.

## Access MinIOInstance via Service

Add an [external service](https://kubernetes.io/docs/concepts/services-networking/service/) in MinIOInstance definition to enable Service based access to the MinIOInstance pods. Refer [the example here](https://github.com/minio/minio-operator/blob/master/examples/minioinstance-with-external-service.yaml?raw=true) for details on how to setup service based access for MinIOInstance pods.
