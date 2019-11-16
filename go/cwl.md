# [Introduction]( https://www.commonwl.org/user_guide/01-introduction/index.html )

cwl是用于描述命令行工具并将它们联系在一起以创建工作流的一种方式。它是一种规范而非一种特殊的软件、使用cwl描述的工具和工作流可以在支持cwl标准的平台上进行移植

# [start]( https://www.commonwl.org/user_guide/02-1st-example/index.html )

```yaml
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: echo
inputs:
  message:
    type: string
    inputBinding:
      position: 1
outputs: []
```

```yaml
#echo-job.yml
message: Hello world!
```

```shell
$ cwl-runner 1st-tool.cwl echo-job.yml
[job 1st-tool.cwl] /tmp/tmpmM5S_1$ echo \
    'Hello world!'
Hello world!
[job 1st-tool.cwl] completed success
{}
Final process status is success

```

* The `baseCommand` provides the name of program that will actually run (`echo`). `echo` is a built-in program in the bash and C shells.
* The `inputs` section describes the inputs of the tool. This is a mapped list of input parameters (see the [YAML Guide](https://www.commonwl.org/user_guide/yaml/#maps) for more about the format) and each parameter includes an identifier, a data type, and optionally an `inputBinding`. The `inputBinding` describes how this input parameter should appear on the command line. In this example, the `position` field indicates where it should appear on the command line.

Key Points 

* CWL documents are written in YAML and/or JSON.
* The command called is specified with `baseCommand`.
* Each expected input is described in the `inputs` section.
* Input values are specified in a separate YAML file.
* The tool description and input files are provided as arguments to a CWL runner.