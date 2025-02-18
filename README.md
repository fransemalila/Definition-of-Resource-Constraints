# Kubernetes Resource Quotas Example

This document demonstrates how to use Resource Quotas in Kubernetes to limit the total amount of resources consumed within a namespace.  Resource Quotas allow cluster administrators to restrict CPU, memory, storage, and the number of objects (pods, services, etc.) created in a namespace.

## Prerequisites

* A working Kubernetes cluster.
* `kubectl` command-line tool configured to connect to your cluster.

## Step 1: Creating a ResourceQuota

1.  **Create a `resource-quota.yaml` file with the following content:**

    This ResourceQuota, named `object-counts`, will be created in the `default` namespace. It limits the number of pods to 4 and the total memory usage of all pods to 5Gi.

    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: object-counts
      namespace: default
    spec:
      hard:
        pods: 4
        limits.memory: 5Gi
    ```

2.  **Apply the ResourceQuota definition:**

    ```bash
    kubectl apply -f resource-quota.yaml
    ```

3.  **Describe the ResourceQuota to view its details:**

    ```bash
    kubectl describe resourcequota object-counts
    ```

    The output will show the defined limits and current usage for each resource.  Initially, the usage will likely be 0 for both `pods` and `limits.memory`.

## Step 2: Rejecting Pod Creation Due to Resource Quota Limits

1.  **Create a `pod-rejected.yaml` file with the following content:**

    This pod definition requests 6Gi of memory. Since the ResourceQuota in the `default` namespace limits total memory usage to 5Gi, creating this pod will exceed the quota.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: random-generator
    spec:
      containers:
      - image: k8spatterns/random-generator:1.0
        name: random-generator
        resources:
          requests:
            cpu: 100m
            memory: 6Gi
          limits:
            memory: 6Gi
    ```

2.  **Attempt to apply the pod definition:**

    ```bash
    kubectl apply -f pod-rejected.yaml
    ```

    You will see an error message indicating that the pod creation was rejected because it would exceed the `limits.memory` quota. The error message will be similar to:

    ```
    Error from server (Forbidden): error when creating "pod-rejected.yaml": pods "random-generator" is forbidden: exceeded quota: object-counts, requested: limits.memory=6Gi, used: limits.memory=0, limited: limits.memory=5Gi
    ```

## Step 3: Allowing Pod Creation Within Resource Quota Limits

1.  **Create a `pod-allowed.yaml` file with the following content:**

    This pod definition requests 2Gi of memory. This is within the 5Gi limit defined by the ResourceQuota.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: random-generator
    spec:
      containers:
      - image: k8spatterns/random-generator:1.0
        name: random-generator
        resources:
          requests:
            cpu: 100m
            memory: 2Gi
          limits:
            memory: 2Gi
    ```

2.  **Apply the pod definition:**

    ```bash
    kubectl apply -f pod-allowed.yaml
    ```

    This time, the pod will be created successfully because its resource requests fall within the ResourceQuota limits.

3.  **Describe the ResourceQuota again to observe the updated usage:**

    ```bash
    kubectl describe resourcequota object-counts
    ```

    The output will now show the `limits.memory` usage reflecting the resources requested by the `random-generator` pod (2Gi).  The `pods` usage will increment as well.

## Explanation of Resource Quotas

*   **ResourceQuota:** A Kubernetes object that defines limits on the resources that can be consumed within a namespace.
*   **`spec.hard`:** Defines the hard limits for each resource.  The scheduler will prevent the creation of any objects that would cause these limits to be exceeded.
*   **Supported Resources:** ResourceQuotas support limits on a variety of resources, including:
    *   `pods`:  Total number of pods.
    *   `limits.cpu`: Total CPU limits for all containers.
    *   `limits.memory`: Total memory limits for all containers.
    *   `requests.cpu`: Total CPU requests for all containers.
    *   `requests.memory`: Total memory requests for all containers.
    *   `count/<resource>`: Total count of specific resources (e.g., `count/persistentvolumeclaims`, `count/services`).
*   **Namespaces:** ResourceQuotas are namespaced, meaning they apply only to the namespace in which they are created.

## Use Cases

Resource Quotas are essential for:

*   **Resource Allocation:**  Fairly allocating resources among different teams or applications in a shared cluster.
*   **Cost Management:**  Controlling the total resource consumption to manage infrastructure costs.
*   **Preventing Resource Starvation:** Protecting the cluster from a single application consuming all available resources.
*   **Security:**  Limiting the potential impact of compromised applications by restricting their resource usage.