pipeline {
  agent any

  environment {
    AWS_REGION= "us-east-1"
    AWS_ACCESS_KEY_ID= credentials('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY= credentials('AWS_SECRET_ACCESS_KEY')
    AWS_SESSION_TOKEN= credentials('AWS_SESSION_TOKEN')
    SONARQUBE_ACCESS_TOKEN="c316609771a51028089adec4949d002effb3756b"
    SONARQUBE_URL="http://ec2-54-158-56-66.compute-1.amazonaws.com:81"
    OWASP_ZAP_URL="http://ec2-54-158-56-66.compute-1.amazonaws.com"
    OWASP_ZAP_API_KEY="uposfjoipjqnobfp"
    APPLICATION_URL="http://javavulnerableapp-env.eba-vfzpqfrv.us-east-1.elasticbeanstalk.com"
  }
  stages {
    // stage('codeguru') {
    //   steps{
    //         sh '''#!/bin/bash
    //         /usr/local/bin/aws sts get-caller-identity
    //         echo $GIT_COMMIT
    //         CODEGURU_REPO_ARN=$(/usr/local/bin/aws codeguru-reviewer list-repository-associations --region $AWS_REGION | jq '.RepositoryAssociationSummaries[] | select(.Name | startswith("vulnerable-java-app")).AssociationArn' | tr -d '[\"\n]')
    //         CODEGURU_REVIEW_ARN=$(/usr/local/bin/aws codeguru-reviewer create-code-review --region $AWS_REGION --name mycodereview-$GIT_COMMIT --repository-association-arn $CODEGURU_REPO_ARN --type '{"RepositoryAnalysis": {"RepositoryHead": {"BranchName": "main"}},"AnalysisTypes": ["CodeQuality"]}' | jq '.CodeReview.CodeReviewArn' | tr -d '[\"\n]')
    //         /usr/local/bin/aws codeguru-reviewer wait code-review-completed --code-review-arn $CODEGURU_REVIEW_ARN --region $AWS_REGION
    //         '''
    //   }
    // }
    stage('Sonarqube-scanner') {
      steps{
          sh '''#!/bin/bash -e
          /opt/sonar-scanner/bin/sonar-scanner -X -Dsonar.sources=. -Dproject.settings=sonar-project.properties -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_ACCESS_TOKEN > sonarqube_scanreport.json
          aws s3 cp sonarqube_scanreport.json s3://dsop-bucket-1234567890/
          curl -u $SONARQUBE_ACCESS_TOKEN: -G --data-urlencode "branch=master" --data-urlencode "projectKey=normaljavarepo" $SONARQUBE_URL/api/qualitygates/project_status > result.json
          if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $echo "Aborting because of an error high risk dependencies..." && exit 1;fi
          echo "build stage completed"
          '''
      }
    }
    stage('dependency-check-scan') {
        steps{
            sh '''#!/bin/bash -e
            pwd
            sudo /opt/dependency-check/bin/dependency-check.sh --project "java" --format JSON --scan .
            echo "OWASP dependency check analysis status is completed..."; 
            ls -lrt && pwd && cat dependency-check-report.json
            sudo chmod 775 *
            aws s3 cp dependency-check-report.json s3://dsop-bucket-1234567890/
            '''
        }
    }
    stage('report-analysis') {
        steps{
            sh '''#!/bin/bash -e
            #jq "{ \\"messageType\\": \\"CodeScanReport\\", \\"reportType\\": \\"OWASP-Dependency-Check\\", \
            \\"createdAt\\": $(date +\\"%Y-%m-%dT%H:%M:%S.%3NZ\\"), \\"source_repository\\": env.CODEBUILD_SOURCE_REPO_URL, \
            \\"source_branch\\": env.CODEBUILD_SOURCE_VERSION, \
            \\"build_id\\": env.CODEBUILD_BUILD_ID, \
            \\"source_commitid\\": env.CODEBUILD_RESOLVED_SOURCE_VERSION, \
            \\"report\\": . }" dependency-check-report.json > payload.json
         '''
      }
    }
    stage('Run script') {
        steps{
            sh '''#!/bin/bash -e
            python3 zap.py
         '''
        }
    }
  }
}

            // if( cat dependency-check-report.json | grep -i HIGHEST); 
            // then
            //   echo "Aborting because of high risk dependencies..." && exit 1; 
            // fi

              // aws sns publish --region us-east-1 --topic-arn \"arn:aws:sns:us-east-1:163112212549:Jenkins\" --message-structure json  --message file://dependency-check-report.json
         
          // curl http://ec2-54-158-56-66.compute-1.amazonaws.com:81/api/qualitygates/project_status?projectKey=normaljavarepo >result.json

// http://ec2-54-158-56-66.compute-1.amazonaws.com:81/api/qualitygates/project_status?analysesId=[ID]

// http://ec2-54-158-56-66.compute-1.amazonaws.com:81/api/qualitygates/project_status?projectKey=normaljavarepo

// curl -u c316609771a51028089adec4949d002effb3756b: -G --data-urlencode "branch=master" --data-urlencode "projectKey=normaljavarepo" http://ec2-54-158-56-66.compute-1.amazonaws.com:81/api/qualitygates/project_status > result.json

// curl -u $SONARQUBE_ACCESS_TOKEN: -G --data-urlencode "branch=master" --data-urlencode "projectKey=normaljavarepo" $SONARQUBE_URL/api/qualitygates/project_status > result.json

// export SONARQUBE_ACCESS_TOKEN="c316609771a51028089adec4949d002effb3756b"
// export SONARQUBE_URL="http://ec2-54-158-56-66.compute-1.amazonaws.com:81"
