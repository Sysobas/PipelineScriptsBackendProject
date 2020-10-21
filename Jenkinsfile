//timestamps{
    node('nodejs'){
        stage('Checkout'){
            //checkout([$class: 'GitSCM', branches: [[name: '*/openshift']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/cmotta2016/PipelineScriptsBackendProject.git']]])
            checkout scm
        }
        stage('Compile'){
            sh 'npm set registry http://cicdtools.oracle.msdigital.pro:8081/repository/npm-group'
            sh 'npm install'
            sh 'rm -rf teste-build.tgz > /dev/null 2>&1'
            sh 'tar czvf teste-build.tgz * --exclude node_modules'
        }

        openshift.withCluster() {
            openshift.withProject("cicd") {
                    def jobTemplate = readFile(file:'depcheck_job_scan.yaml')
                    //openshift.delete(jobTemplate)
                    def jobCreated = openshift.create(jobTemplate)
                    def job = openshift.selector("job", "node-backend-v1-depcheck")
                    timeout(5) {
                        job.untilEach(1) {
                            return (it.related('pods').object().status.phase != "Pending")
                        }
                    }
                    job.logs('-f')
                    def obj = job.related('pods').object()
                    if (obj.status.phase in ["Error", "Failed", "Cancelled"]) {
                        error(obj.toString())
                    }
                }
        }//withCluster
    }//node
//}//timestamps