update_settings(max_parallel_updates=16)

real_resources=['fake-kube-apiserver', 'fake-kube-controller-manager', 'fake-kube-proxy', 'fake-kube-scheduler']

BAZEL_RUN_CMD = "bazel run --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 %s %s"

BAZEL_SOURCES_CMD = """
  bazel query 'filter("^//", kind("source file", deps(set(%s))))' --order_output=no
  """.strip()

BAZEL_BUILDFILES_CMD = """
  bazel query 'filter("^//", buildfiles(deps(set(%s))))' --order_output=no
  """.strip()

def escape_target(target):
  return target.replace("//", "").replace("/", "_").replace(":", "-")

def build_deps_file(target):
  return ".tilt_builddeps_{}".format(escape_target(target))

def source_deps_file(target):
  return ".tilt_sourcedeps_{}".format(escape_target(target))

def bazel_labels_to_files(labels):
  files = {}
  for l in labels:
    if l.startswith("@"):
      continue
    if l.startswith("//external/") or l.startswith("//external:"):
      continue
    elif l.startswith("//"):
      l = l[2:]

    path = l.replace(":", "/")
    if path.startswith("/"):
      path = path[1:]

    files[path] = None

  return files.keys()

def bazel_k8s(target):
  build_deps_for_target(target)  # this takes care of watching
  source_deps_for_target(target)  # this takes care of watching

  return local("bazel run %s" % target)

def bazel_build(image, target, options=''):
  build_deps_for_target(target)  # this takes care of watching

  custom_build(
    image,
    BAZEL_RUN_CMD % (target, options),
    source_deps_for_target(target),
    tag="image",
    match_in_env_vars=True,
    # TODO: get ignores right
    ignore=['.tilt_depslist_cmd_kube-apiserver']
  )

def build_deps_for_target(target):
  file = build_deps_file(target)
  local('test -f {file} || touch {file}'.format(file=file), quiet=True, echo_off=True)

  cmd = BAZEL_BUILDFILES_CMD % target
  local_resource('build-deps-{}'.format(escape_target(target)),
                 '{} > {}'.format(cmd, file),
                 resource_deps=real_resources,
                 allow_parallel=True,
                 )

  deps = bazel_labels_to_files(str(read_file(file)).splitlines())
  if not deps:
    deps = ['.dummy']

  return deps


def source_deps_for_target(target):
  file = source_deps_file(target)
  local('test -f {file} || touch {file}'.format(file=file), quiet=True, echo_off=True)

  cmd = BAZEL_SOURCES_CMD % target
  local_resource('source-deps-{}'.format(escape_target(target)),
                 '{} > {}'.format(cmd, file),
                 resource_deps=real_resources,
                 allow_parallel=True,
                 )

  deps = bazel_labels_to_files(str(read_file(file)).splitlines())
  if not deps:
    deps = ['.dummys']

  return deps

k8s_yaml(bazel_k8s("//tilt:apiserver"))
k8s_yaml(bazel_k8s("//tilt:controller-manager"))
k8s_yaml(bazel_k8s("//tilt:proxy"))
k8s_yaml(bazel_k8s("//tilt:scheduler"))

bazel_build("bazel/cmd/kube-apiserver", "//cmd/kube-apiserver:image", "-- --norun")
bazel_build("bazel/cmd/kube-controller-manager", "//cmd/kube-controller-manager:image", "-- --norun")
bazel_build("bazel/cmd/kube-proxy", "//cmd/kube-proxy:image", "-- --norun")
bazel_build("bazel/cmd/kube-scheduler", "//cmd/kube-scheduler:image", "-- --norun")
