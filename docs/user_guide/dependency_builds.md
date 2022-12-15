# Chained Builds With Dependency Tracking

You can split image build directives over multiple files and chain them.

If the `prerequisites` list is present under the `config` key, stacker will
make sure to build all the layers in the stacker.yaml files found at the paths
contained in the list. This way stacker supports building multiple
stacker.yaml files in the correct order.

In this particular case the parent folder of the current folder, let's call it
`parent`, has 3 subfolders `folder1`, `folder2` and `folder3`, each containing a
`stacker.yaml` file. The example `config` above is in `parent/folder1/stacker.yaml`.

When `stacker build -f parent/folder1/stacker.yaml` is invoked, stacker would search
for the other two stacker.yaml files and build them first, before building
the stacker.yaml specified in the command line.
