genrule(
  name = "gen-foo",
  outs = ["foo.cc"],
  cmd = "echo 'int main() { return 42; }' > $@",
)

cc_binary(
  name = "foo",
  srcs = [":foo.cc"],
)
