#!/usr/bin/groovy
node{

  def devopsPipeline
  def forgePipeline
  def stagedDevopsProject
  def stagedForgeProject
  def pipelineForgePrId

  stage 'Update fabric8-devops deps'
  ws {
    git 'https://github.com/fabric8io/fabric8-devops.git'
    sh "git remote set-url origin git@github.com:fabric8io/fabric8-devops.git"
    devopsPipeline = load 'release.groovy'
    def devopsPipelinePrId = devopsPipeline.updateDependencies('http://central.maven.org/maven2/')

    stage 'Stage fabric8-devops'
    stagedDevopsProject = devopsPipeline.stage()
  }

  stage 'Update fabric8-forge deps'
  ws {
    git 'https://github.com/fabric8io/fabric8-forge.git'
    sh "git remote set-url origin git@github.com:fabric8io/fabric8-forge.git"
    forgePipeline = load 'release.groovy'
    pipelineForgePrId = forgePipeline.updateDependencies('https://oss.sonatype.org/content/repositories/staging/')

    stage 'Stage fabric8-forge'
    stagedForgeProject = forgePipeline.stage()
  }

  //stage 'Deploy pipeline-test-project'
  //forgePipeline.deploy(stagedForgeProject)

  stage 'Approve release'
  forgePipeline.approveRelease(stagedForgeProject)

  stage 'Promote fabric8-devops'
  ws {
    devopsPipeline.release(stagedDevopsProject)
    if (devopsPipelinePrId != null){
      devopsPipeline.mergePullRequest(devopsPipelinePrId)
    }
  }

  stage 'Promote fabric8-devops'
  ws{
    forgePipeline.release(stagedForgeProject)
    if (pipelineForgePrId != null){
      forgePipeline.mergePullRequest(pipelineForgePrId)
    }
  }
}