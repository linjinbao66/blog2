---

title: mvn-generate-project
date: 2022-08-16
tags: ["maven", "mvn", "java", "cli"]

---

use mvn cli to generate a project:

```shell

mvn archetype:generate -DgroupId=tk.amrom -DartifactId=proto-java -DinteractiveMode=false -DoutputDirectory=./

```

explain:

1. use plugin archetype:generate 
2. require args groutId and artifactId and DinteractiveMode
