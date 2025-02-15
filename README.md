Issue retaining intermediate files with newer bazels 
====================================================

Consider the following [`BUILD`](./BUILD) file:

```Python
genrule(
  name = "gen-foo",
  outs = ["foo.cc"],
  cmd = "echo 'int main() { return 42; }' > $@",
)

cc_binary(
  name = "foo",
  srcs = [":foo.cc"],
)
```

When building `:foo`, the file foo.cc file is generated as input to build
the binary.

After building, we expect the `foo` binary and the `foo.cc` file to be in our
generated outputs.

This works correctly and as expected with Bazel-6

```
bazel-6.5.0 clean --expunge ; bazel-6.5.0 build :foo ; ls -l bazel-bin/foo.cc bazel-bin/foo
-r-xr-xr-x 1 user group 7608 Jan 29 10:28 bazel-bin/foo
-r-xr-xr-x 1 user group   26 Jan 29 10:28 bazel-bin/foo.cc
```

However, with newer bazel, bazel tries to be 'smart' and hide the intermediate
results, so `foo.cc` is not visible:

```
bazel-8.0.1 clean --expunge ; bazel-8.0.1 build :foo ; ls -l bazel-bin/foo.cc bazel-bin/foo
# ...
ls: cannot access 'bazel-bin/foo.cc': No such file or directory
-r-xr-xr-x 1 user group 7608 Jan 29 10:29 bazel-bin/foo
```

This of course is a major headache: now any tooling such as editors will not
be able to see and index generated files, making the default behavior not
desirable.

Is there a flag that re-enables the correct behavior in newer bazels ?

## Updates

  * Running with `-s` shows nicely how the file is generated, but it
    does not show up afterwards.
  * this only seems to happen with a remote cache enabled and the
    second time it is executed. So the file is there possibly remotely,
    and bazel does not bother copying it back locally. Which is problematic,
    as it means this only behaves correctly if the remote cache
    is disabled.

## FIX

When a remote cache is enabled, bazel elects only to download the artifacts
requested, not any intermediate outputs.

https://blog.bazel.build/2023/12/11/bazel-7-release.html#build-without-the-bytes-bwob

This can be re-enabled by setting flag `--remote_download_outputs=all`

