# This repository is **DEPRECATED**
Use the ZAP implementation found [HERE](https://github.com/redhat-cop/containers-quickstarts/tree/master/jenkins-agents/jenkins-agent-zap)

## OWASP [ZAP](https://github.com/zaproxy/zaproxy) Image For OpenShift

## Overview
The public docker registry version of OWASP's Zed Attack Proxy (ZAP) is not
compatible with OpenShift without using privleged containers. This
Docker image resolves that issue.

## Running

The semantics of running this are identical to the public OWASP ZAP docker
image, so look at the Wiki page [HERE](https://github.com/zaproxy/zaproxy/wiki/Docker).

## Deploying In OpenShift
```bash
oc new-build -l 'role=jenkins-slave' https://github.com/rht-labs/owasp-zap-openshift.git
```

## Configuring In OpenShift Jenkins
1. Log in to Jenkins with an account which has permissions to manage the Jenkins instance
1. Install the following plugins: 
   1. HTML Publisher Plugin
1. Restart Jenkins
1. Log back in to Jenkins and navigate to `Manage Jenkins -> Configure System`
1. Scroll down to the `Kubernetes` cloud configuration
1. Add a new "Pod Template" as shown below: ![KubePodTemplate](Kube_Pod_Template.png)


## Using it in your Jenkinsfile
```groovy

stage('Get a ZAP Pod') {
    node('zap') {
        stage('Scan Web Application') {
            dir('/zap') {
                def retVal = sh returnStatus: true, script: '/zap/zap-baseline.py -r baseline.html -t http://<some-web-site>'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: '/zap/wrk', reportFiles: 'baseline.html', reportName: 'ZAP Baseline Scan', reportTitles: 'ZAP Baseline Scan'])
                echo "Return value is: ${retVal}"
            }
        }
    }
}
```

## Use it in Jenkinsfile for usage with Sonarqube ZAP plugin (specifically -x report, no directory, no special chars in name)
```groovy

stage('Get a ZAP Pod') {
    node('zap') {
        stage('Scan Web Application') {
          sh "/zap/zap-baseline.py -d -m 5 -x zaprpt.xml -t http://<some-web-site>"
//no mvn, so stash it and unstash later in pipeline on a maven node instead of ZAP node... 
//sh "mvn sonar:sonar -Dsonar.zaproxy.reportPath=/zap/wrk/zaprpt.xml"
          stash name: "zaproxyreport", includes: "/zap/wrk/zaprpt.xml"
        }
    }
}
```




stash/copy/pull the file from:
/zap/wrk/zaprpt.xml

