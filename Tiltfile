
BAZEL_RUN_CMD = "bazel run --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 %s %s"

BAZEL_SOURCES_CMD = """
  bazel query 'filter("^//", kind("source file", deps(set(%s))))' --order_output=no
  """.strip()

BAZEL_BUILDFILES_CMD = """
  bazel query 'filter("^//", buildfiles(deps(set(%s))))' --order_output=no
  """.strip()

def bazel_labels_to_files(labels):
  files = {}
  for l in labels:
    if l.startswith("//external/") or l.startswith("//external:"):
      continue
    elif l.startswith("//"):
      l = l[2:]

    path = l.replace(":", "/")
    if path.startswith("/"):
      path = path[1:]

    files[path] = None

  return files.keys()

def watch_labels(labels):
  watched_files = []
  for l in labels:
    if l.startswith("@"):
      continue
    elif l.startswith("//external/") or l.startswith("//external:"):
      continue
    elif l.startswith("//"):
      l = l[2:]

    path = l.replace(":", "/")
    if path.startswith("/"):
      path = path[1:]

    watch_file(path)
    watched_files.append(path)

  return watched_files

def bazel_k8s(target):
  build_deps = str(local(BAZEL_BUILDFILES_CMD % target)).splitlines()
  source_deps = str(local(BAZEL_SOURCES_CMD % target)).splitlines()
  watch_labels(build_deps)
  watch_labels(source_deps)

  return local("bazel run %s" % target)

def bazel_build(image, target, options=''):
  build_deps = str(local(BAZEL_BUILDFILES_CMD % target)).splitlines()
  watch_labels(build_deps)

  source_deps = str(local(BAZEL_SOURCES_CMD % target)).splitlines()
  source_deps_files = bazel_labels_to_files(source_deps)

  custom_build(
    image,
    BAZEL_RUN_CMD % (target, options),
    source_deps_files,
    tag="image",
    match_in_env_vars=True,
  )

k8s_yaml(bazel_k8s("//tilt:apiserver"))
k8s_yaml(bazel_k8s("//tilt:controller-manager"))
k8s_yaml(bazel_k8s("//tilt:proxy"))
k8s_yaml(bazel_k8s("//tilt:scheduler"))

bazel_build("bazel/cmd/kube-apiserver", "//cmd/kube-apiserver:image", "-- --norun")
bazel_build("bazel/cmd/kube-controller-manager", "//cmd/kube-controller-manager:image", "-- --norun")
bazel_build("bazel/cmd/kube-proxy", "//cmd/kube-proxy:image", "-- --norun")
bazel_build("bazel/cmd/kube-scheduler", "//cmd/kube-scheduler:image", "-- --norun")
