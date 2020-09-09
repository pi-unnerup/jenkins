env = System.getenv()

// projects

def projects = [
  [
    name       : "netassert",
    branch     : "**",
    jenkinsfile: "Jenkinsfile",
    pollTrigger: "* * * * *",
    cronTrigger: ""
  ],
  [
    name         : "dsl-jenkins",
    branch     : "**",
    jenkinsfile: "Jenkinsfile",
    pollTrigger: "* * * * *",
    cronTrigger: ""
  ]
]

def pipelineDefaults = [
  depth                : 1,
  environmentWhitelist : [],
  // options relevant to pipelineJobParameters
  image                : "latest",
  disabled             : false,
  jobParameters        : {},
  allowConcurrentBuilds: true,
  // these use java regular expression pattern matching http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html
  gitPathIncluded      : "",
  gitPathExcluded      : ""
]

for (pipeline in projects) {
  for (pipelineDefault in pipelineDefaults) {
    if (pipeline[pipelineDefault.key] == null) {
      pipeline[pipelineDefault.key] = pipelineDefault.value
    }
  }
}

print "\n\nProjects:\n\n${projects}\n\n"

// ---

def getJobName(project) {
  return "${project.jobName ? project.jobName : project.name}"
}

// ---

def getJenkinsfileName(pipeline, stage = '') {
  if (pipeline.jenkinsfile) {
    fileName = pipeline.jenkinsfile
  } else {
    fileName = 'Jenkinsfile'
  }
  return fileName + (stage ? ".${stage}" : '')
}

def getBranch(pipeline = "") {
  if (isLocalJobOverride() || pipeline == "") {
    return '**'
  }
  if (pipeline.branch) {
    return pipeline.branch
  }
  return '*/master'
}

def getPollTrigger(pipeline = "") {
  if (isInTest() || pipeline == "") {
    return ''
  }
  if (pipeline.pollTrigger || pipeline.pollTrigger == "") {
    return pipeline.pollTrigger
  }
  return "* * * * *"
}

def getCronTrigger(pipeline = "") {
  if (isInTest() || pipeline == "") {
    return ''
  }
  if (pipeline.cronTrigger || pipeline.cronTrigger == "") {
    return pipeline.cronTrigger
  }
  return ""
}

// ------------

defaultListColumns = {
  status()
  weather()
  name()
  lastSuccess()
  lastFailure()
  lastDuration()
  lastBuildConsole()
  progressBar()
  buildParameters()
  buildButton()
}

projects.each { project ->

  jobName = "${getJobName(project)}"
  pipelineName = "${project.name}"
  if (pipelineName != jobName) {
    pipelineName += " | ${jobName}"
  }

  print "Debug: pipelineName: ${pipelineName} | jobName: ${jobName}\n"

  pipelineJobParametersDefault = {}
  pipelineJobParametersCustom = project.jobParameters

  pipelineJobParameters = pipelineJobParametersDefault << pipelineJobParametersCustom

  isParameterisedPipeline = true
  try {
    if (!pipelineJobParameters()) {
      isParameterisedPipeline = false
    }
  } catch (x) {
  }

  scmParameters = {
    git {
      remote {
        url getRepoUrl(project)
        credentials("jenkins_credentials")
      }
      branch(getBranch(project))
      extensions {
        cleanBeforeCheckout()
      }
      configure { gitScm ->
        gitScm / 'extensions' << 'hudson.plugins.git.extensions.impl.PathRestriction' {
          includedRegions(project.gitPathIncluded)
          excludedRegions(project.gitPathExcluded)
        }
      }
    }
  }

  pipelineJob(jobName) {
    if (isParameterisedPipeline) {
      parameters pipelineJobParameters
    } else {
      configure { thisProject ->
        (thisProject / 'properties').remove(thisProject / 'properties' / 'hudson.model.ParametersDefinitionProperty')
      }
    }

    configure {
      (it / 'concurrentBuild').setValue(project.allowConcurrentBuilds)
    }

    description "Build ${jobName}"
    disabled(project.disabled)
    definition {
      cpsScm {
        scm scmParameters
        scriptPath(getJenkinsfileName(project))
      }
      triggers {
        scm(getPollTrigger(project))
        cron(getCronTrigger(project))
      }
      logRotator(-1, 20, -1, -1)
      quietPeriod(0)
    }
  }
}