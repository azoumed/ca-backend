@Library("jenkins-pipeline-library@fix/set_projectBranch")

import fr.creative.jenkins.BuildContextHolder
import fr.creative.jenkins.config.ConfigUtils
import fr.creative.jenkins.exception.TechnicalException
import fr.creative.jenkins.model.DockerRegistry
import fr.creative.jenkins.model.Module
import fr.creative.jenkins.model.Sonar
import fr.creative.jenkins.service.*
import fr.creative.jenkins.utils.*

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

node() {
    // Nom du Projet Gitlab
    String projectName = "cda-api"
    // Branche GIT à builder/packager/déployer
    String projectBranch = params.BRANCH_NAME
    // Environnement sur lequel on souhaite déployer l'application
    String deployTo = params.DEPLOY_TO
    // Version de l'application que l'on souhaite déployer
    String version = params.PROJECT_VERSION
    // Skip Deploy
    boolean skipBuild = params.SKIP_BUILD
    // Skip Sonarqube
    boolean skipSonarqube = params.SKIP_SONARQUBE
     // set critical Sonarqube behaviour
    boolean isSonarqubeCritical = true
    // Skip Deploy
    boolean skipPackage = params.SKIP_PACKAGE
    // Skip Publish
    boolean skipPublish = params.SKIP_PUBLISH
    // Skip Publish
    boolean skipDeploy = params.SKIP_DEPLOY
    // Skip Scan
    boolean skipScan = params.SKIP_SCAN

    boolean isMergeRequest = false

    // URL Channel Teams
    String webhookUrl = "https://creativecorebusiness.webhook.office.com/webhookb2/3757b4c6-ab34-424d-8289-0ea5d29282bc@07cdf6c2-b866-4ffc-a7cf-eaeb75545f95/JenkinsCI/8a1c571231184f73b10b5f85519a528d/41524f31-8514-413e-8217-a9a6bad73477"

    try {
        BuildContextHolder.init(this)

        // Ne pas deployer s'il s'agit d'une pipeline déclenché par gitlab
        if(GitlabService.instance().isTriggeredByGitlab()){
          skipDeploy = true
          skipScan = true
          isSonarqubeCritical = false
          isMergeRequest = true

          projectBranch = "origin/" + env.gitlabSourceBranch
        }

        GitlabService.instance().updatePipelineStatusToRunning()

        stage('Prepare') {
            checkout scm
            sh("chmod 777 -R .")
            sh("find . -type f -print0 | xargs -0 dos2unix")
            if (isMergeRequest) {
              JenkinsService.instance().appendBuildDescription("'${projectName}': Merge Request Branch '${projectBranch}' To 'origin/develop'")
            } else {
              JenkinsService.instance().appendBuildDescription("'${projectName}': Deploy Branch '${projectBranch}' To '${deployTo}'")
            }
        }

        stage("Build") {
            if (!skipBuild) {
                println "Building project"
                sh "bash .platforms/ci/build.sh"
            }
        }

        stage("TU") {
            if (!skipBuild) {
                println "Building project"
                sh "bash .platforms/ci/test.sh"
            }
        }

        stage("Security Tests (Trivy)") {
            boolean trivyError=false
            try {
                // Rapport au format HTML
                sh("bash .platforms/ci/trivy.sh")
            } catch (err) {
                trivyError=true
            } finally {
                // Publication du rapport HTML
                JenkinsService.instance().publishHtml("./.trivy/", "TrivyReport")
                archiveArtifacts artifacts: '.trivy/trivy.html', excludes: null
                if (trivyError) {
                    qualityWarningStage += " TRIVY |"
                    String url = JenkinsService.instance().getJobUrl() + "/TrivyReport/"
                    JenkinsService.instance().setStageAsUnstable("Trivy - Niveau de sécurité insuffisant > ${url}")
                }
            }
        }


        stage("Package") {
            if (!skipPackage) {
                DockerService.instance().login(DockerRegistry.getDefaultRegistry())
                println "Creating Docker Images"
                if (isMergeRequest) {
                  sh "bash .platforms/ci/package.sh"
                } else {
                  sh "bash .platforms/ci/package.sh --with-push"
                }
            }
        }

        stage("Sonarqube") {
            if (!skipSonarqube) {
                echo "Qualimetry Sonarqube"
                sh "bash .platforms/ci/sonar.sh"

                // Validation de l'analyse sonar (blocage si bug critique)
                def sonarproject = new Sonar()
                sonarproject.key = "cda-api"
                try {
                SonarService.instance().checkProjectStatus(sonarproject, projectBranch, true)
            } catch (err) {
                qualityWarningStage += " SONARQUBE |"
                JenkinsService.instance().setStageAsUnstable("Sonarqube - Niveau de qualité du code insuffisant, merci de consulter le rapport et de corriger les erreurs")
            }
            }
        }

        stage("Publish") {
            if (!skipPublish) {
                DockerService.instance().login(DockerRegistry.getDefaultRegistry())
                echo "publish docker image in nexus"
                sh "bash .platforms/ci/publish.sh"
            }
        }

        stage("Deploy") {
            if (!skipDeploy ) {
              def config = jsonParse(new File("${env.WORKSPACE}/src/git-version.json").getText())
              version = config.imageversion
              withKubeConfig([credentialsId: 'kubeconfig-gcc-admin']) {
                println "Deploying project using generated version ${version}"
                sh "bash .platforms/k8s/deploy.sh ${deployTo} ${version}"
                JenkinsService.instance().appendBuildDescription("Deploying version ${version} To ${deployTo}")
              }
            }
        }

        if (isMergeRequest) {
          println "'${projectName}': Successful merge request"
        } else {
          println "'${projectName}': Successful project deployment"
        }
        //MicrosoftTeamsService.instance().send(webhookUrl, "Deploy 'TunnelCourtier' (Branch '${projectBranch}' To '${deployTo}') SUCCESS")
        GitlabService.instance().updatePipelineStatusToSuccess()
    } catch (err) {
      JenkinsService.instance().raiseTechnicalError(err)
      //MicrosoftTeamsService.instance().send(webhookUrl, "Deploy 'TunnelCourtier' (Branch '${projectBranch}' To '${deployTo}') FAILED")
      GitlabService.instance().updatePipelineStatusToFailed()
    }
}
