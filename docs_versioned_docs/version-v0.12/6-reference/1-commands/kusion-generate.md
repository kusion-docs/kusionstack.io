# kusion generate

Generate and print the resulting Spec resources of target Stack

### Synopsis

This command generates Spec resources with given values, then write the resulting Spec resources to specific output file or stdout.

 The nearest parent folder containing a stack.yaml file is loaded from the project in the current directory.

```
kusion generate [flags]
```

### Examples

```
  # Generate and write Spec resources to specific output file
  kusion generate -o /tmp/spec.yaml
  
  # Generate spec with custom workspace
  kusion generate -o /tmp/spec.yaml --workspace dev
  
  # Generate spec with specified arguments
  kusion generate -D name=test -D age=18
```

### Options

```
  -D, --argument stringArray   Specify arguments on the command line
      --backend string         The backend to use, supports 'local', 'oss' and 's3'.
  -h, --help                   help for generate
      --no-style               no-style sets to RawOutput mode and disables all of styling
  -o, --output string          File to write generated Spec resources to
  -w, --workdir string         The work directory to run Kusion CLI.
      --workspace string       The name of target workspace to operate in.
```

### Options inherited from parent commands

```
      --profile string          Name of profile to capture. One of (none|cpu|heap|goroutine|threadcreate|block|mutex) (default "none")
      --profile-output string   Name of the file to write the profile to (default "profile.pprof")
```

### SEE ALSO

* [kusion](index.md)	 - Kusion is the Platform Orchestrator of Internal Developer Platform

###### Auto generated by spf13/cobra on 12-Jun-2024