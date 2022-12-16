# Tanzu Developer Tools for Visual Studio

Tanzu Developer Tools for Visual Studio is an official VMware Tanzu IDE extension for Visual Studio
to help you develop with Tanzu Application Platform.

This plug-in extends Microsoft Visual Studio 2022 only. It is incompatible with Visual Studio Code
and Visual Studio for Mac.

> **Note** This extension is currently in the beta stage of development.

## Extension Features

The extension has the following features:

- **See code updates running on-cluster in seconds**
  With the use of Live Update facilitated by Tilt, deploy your workload once, save changes to the code
  and then, seconds later, see those changes reflected in the workload running on the cluster.
  All Live Update output is filtered to its own output pane window within Visual Studio.

- **Debug workloads directly on the cluster**
  Debug your application in a production-like environment by debugging on a Kubernetes cluster of yours
  that has Tanzu Application Platform.
  The similarity of an environment to production relies on keeping dependencies updated, among other
  variables.