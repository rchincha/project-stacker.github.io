# Additing Annotations 

In the stacker yaml file, `annotations` is a user-specified key value map that will be included in the
final OCI image. Note that these annotations are included in the image manifest
itself and not as part of the index.json.

    annotations:
      a.b.c.key: abc_val
      p.q.r.key: pqr_val

While `config` section supports a similar `labels`, it is more pertitent to the
image runtime. On the other hand, `annotations` is intended to be
image-specific metadata aligned with the
[annotations in the image spec](https://github.com/opencontainers/image-spec/blob/main/annotations.md).
