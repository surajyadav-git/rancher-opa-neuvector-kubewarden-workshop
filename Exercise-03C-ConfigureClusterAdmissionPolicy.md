# Excercise-03C - Enforce AdmissionPolicy



`AdmissionPolicy` is a namespace-wide resource. The policy will process only the requests that are targeting the Namespace where the `AdmissionPolicy` is defined. Other than that, there are no functional differences between the `AdmissionPolicy` and `ClusterAdmissionPolicy` resources.

## 03C -1 ) Example 1: Enforce your first policy