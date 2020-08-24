package(default_visibility = ["//visibility:public"])
load("@io_bazel_rules_k8s//k8s:object.bzl", "k8s_object")

k8s_object(
  name = "apiserver",
  kind = "deployment",
  template = ":apiserver.yaml",
)
k8s_object(
  name = "controller-manager",
  kind = "deployment",
  template = ":controller-manager.yaml",
)
k8s_object(
  name = "proxy",
  kind = "deployment",
  template = ":proxy.yaml",
)
k8s_object(
  name = "scheduler",
  kind = "deployment",
  template = ":scheduler.yaml",
)
