  [@rubbish]: https://github.com/rubbish
  [@SethTisue]: https://github.com/SethTisue
  [@gmalouf]: https://github.com/gmalouf
  [@daggerrz]: https://github.com/daggerrz
  [@indrajitr]: https://github.com/indrajitr
  [@TheBizzle]: https://github.com/TheBizzle
  [@n8han]: https://github.com/n8han/
  [@eed3si9n]: https://github.com/eed3si9n
  [ansible]: http://www.ansible.com/
  [nexus]: http://www.sonatype.com/nexus
  [bintray]: https://bintray.com/
  [1]: http://nescala.org/#talk-37
  [2]: https://github.com/Banno/banno-sbt-plugin
  [3]: http://code.technically.us/post/9545154150/local-external-projects-in-sbt
  [4]: https://gist.github.com/daggerrz/9310300
  [5]: https://github.com/sbt/sbt-dirty-money

sbt round table
---------------

Inspired by [Luke (@rubbish)][@rubbish]'s lightning talk [Better Living Through sbt][1] and subsequently opensourced [Banno/banno-sbt-plugin][2], sbt users who use them at work setting gathered at nescala 2014 day 2 unconference (March 2, 2014). Notes were taken by [Eugene (@eed3si9n)][@eed3si9n].

## Questions

- How many github repository to have?
- Are you using multi-project builds?

### [Seth (@SethTisue)][@SethTisue]: Publish to bintray using SHA1 during dev

multiple (github) repos
3 repos to have separete issue tracking.
work flow gets complicated when make changes in all three repos.

[Jason (@TheBizzle)][@TheBizzle] made [a plugin](https://github.com/NetLogo/publish-versioned-plugin) that makes it semi-automatic to publish to [Bintray][bintray] while development with git sha appended to version.

### [Gary (@gmalouf)][@gmalouf]: Don't use sbt as deployment tool 

"let's keep it to one repo" vs "multiple repo"
=> now 22 repos

try not to use sbt for deployment (writing shell in sbt).
it's not a deployment tool.
we use [Ansible][ansible].

even if we don't opensource, it we take the same approach.

### [Luke (@rubbish)][@rubbish]: Make util plugins for each tech stack

we have several teams.
aws-utils, etc 

we have a cross-cutting thing. common library becomes contention point.

### [Dag (@daggerrz)][@daggerrz]: Modularlize util plugins

that's a smell on modularity. make separate utils for server and client.

all good software devs want to reuse code, so they will start adding things to utils something that might be useful.
having backpressure to implementing to common library actually helps.
some are changed often. others are not.

our pain point is that we are hoping that publishLocal.
(Eugene: Ivy always looks for the last place it grabbed SNAPSHOT; use [sbt-dirty-money][5] to clean the cache for the project) publishLocal should automatically do that.

### [Indrajit (@indrajitr)][@indrajitr]: Source dependency + gitbhu hash

if we there's semantic versioning, sbt allows range in the version.

you can use local file system publish. (but you lose some)

make config-only sbt plugin and use source dep + hash to load it into multiple projects.
sbt won't pick up new commits automatically, so hah is almost required.

### Dag: Use bintray for config sbt plugins

we publishs the binary for the config sbt plugin.
we make custom Build class for different profile like server.

the problem is that you get it wrong 16 times before it works.


### Luke: Use local Sonatype/Artifactory

strongly recommend local [Sonatype Nexus][nexus].
moving to artifactory

### Gary: Transitive dependency issues

we use protobuf and spark.
spark wants protobuf 2.4.1
some library want to use 2.5.

### Luke

we have a library called remote protocol.

### Dag: Excluding transitive dependencies

suppose finagle has xyz deps that want to exclude.
that shows up again in the client-side.
if you use libraryDependency, then it needs to be repeated everywhere.

### Luke: Exlude at ivyXML level

if we exlude at the ivyXML that it's applied after all depedencies are resolved.
I would use ivyXML.

(luke demoing his plugin; log libraries are all exluded)

under target report these resolution cache report.

### Dag

[Remove unwanted deps][4]:

```scala
  val excludedDependencies = Seq(
    "org.slf4j"       -> "slf4j-log4j12",
    "log4j"           -> "log4j",
    "org.jboss.netty" -> "netty" // We have the io.netty dependency
  )
  
  val removeUnwantedDependencies = libraryDependencies ~= { deps =>
    deps.map { d =>
      excludedDependencies.foldLeft(d) { case (acc, (group, artifact)) => acc.exclude(group, artifact) }
    }
  }
```

### Luke: Symlink to other builds

used to use publish SNAPSHOT locally. update on the other library.
but it's too much effort.

inspired by [Nathan (@n8han)][@n8han]'s [Local external projects in sbt][3]:

if I'm working bano-utils and it uses aws-util,
i won't add library depedencies, I add "Project(..., dependencies = symlinkedProjects)" just to coordinate development. It's almost like a multi project build.

### Gary

using sbt 0.12. when I go into build, I have to remember what I did two weeks ago.
It's difficult. A lot of the engineers in build tool.

### Dag

worst part is ivy.
