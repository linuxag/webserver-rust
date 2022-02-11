def project_owner_team_email = 'ashutoshgupta0077@gmail.com'

pipeline
{
    agent
    {
        label 'worker1'
    }
    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}
    parameters { 
    string(name: 'project_owner_team_email', defaultValue: project_owner_team_email, description: 'project_owner_team_email') 
    string(name: 'app_version', defaultValue: 'v1', description: 'app_version') 
    }
    stages
    {
        stage('send-notification')
        {
            
            steps{
                /*script 
                {
                    emailext subject: '${JOB_NAME} - ${BUILD_NUMBER} ', body: 'Job url : ${BUILD_URL}',  to: '${project_owner_team_email}'
                }*/
                sh '''
                echo "mail sent"
                '''
            }
        
        }
        stage('static-analysis-code')
        {
            
            steps{
                 /*script 
                        {
                            withSonarQubeEnv('sonarqube-server') 
                            {
                                def temp_job_name = JOB_BASE_NAME.replaceAll('/','-')
                                def git_branch = 'main' ;
                                def scannerHome = tool 'sonarqube-scanner';

                                sh "${scannerHome}/bin/sonar-scanner -Dsonar.sourceEncoding=UTF-8  -Dsonar.sources=${WORKSPACE} -Dsonar.projectKey=${temp_job_name}-${git_branch} -Dsonar.projectName=${temp_job_name}-${git_branch};"
                                

                                    withCredentials([usernamePassword(credentialsId: 'sonarqube-creds', usernameVariable: 'uname' , passwordVariable: 'upass')]) 
                                    {
                                        sh """
                                        cd .scannerwork
                                        sonar_job_url=\$(cat report-task.txt | grep ceTaskUrl | awk -F 'ceTaskUrl=' '{print \$NF}')
                                        while(true)
                                        do
                                            sonar_job_status=\$(curl -s --user "\$uname:\$upass" \$sonar_job_url | awk -F '"status":' '{print \$NF}' | cut -d ',' -f1 | sed 's/"//g')
                                            sleep 10
                                            if [ "\$sonar_job_status" != 'IN_PROGRESS' ]
                                            then
                                            echo "sonar job completed"
                                            
                                            #get new bugs
                                            new_vulnerabilities=\$(curl -s --user "\$uname:\$upass" http://34.125.209.29:9000/api/measures/search_history?component="${temp_job_name}-${git_branch}"'&'metrics=new_vulnerabilities | awk -F '"value":' '{print \$NF}' | awk -F '}' '{print \$1}' | sed 's/"//g')
                                            new_bugs=\$(curl -s --user "\$uname:\$upass" http://34.125.209.29:9000/api/measures/search_history?component="${temp_job_name}-${git_branch}"'&'metrics=new_bugs | awk -F '"value":' '{print \$NF}' | awk -F '}' '{print \$1}' | sed 's/"//g')
                                            new_violations=\$(curl -s --user "\$uname:\$upass" http://34.125.209.29:9000/api/measures/search_history?component="${temp_job_name}-${git_branch}"'&'metrics=new_violations | awk -F '"value":' '{print \$NF}' | awk -F '}' '{print \$1}' | sed 's/"//g')
                                            
                                            echo "new_vulnerabilities=\$new_vulnerabilities new_bugs=\$new_bugs new_violations=\$new_violations" > /tmp/\${JOB_BASE_NAME}-\${BUILD_ID}.txt
                                            
                                            cat "/tmp/\${JOB_BASE_NAME}-\${BUILD_ID}.txt"
                                            
                                            echo "new_vulnerabilities: \$new_vulnerabilities , new_bugs : \$new_bugs , new_violations : \$new_violations"
                                            exit
                                            fi

                                        done
                                        """
                                    }
                            }

                            def qualitygate = waitForQualityGate()
                            server_report = qualitygate.status
                        }*/
                        sh '''
                        echo "static code analysis using sonarqube success"
                        '''
                    
                }
        
        }
        stage('manual-qualiy-gate')
        {
            
            steps{
                sh '''
                
                echo "send email to required team , email should have link for sonaruqbe and jenkins current job"
                echo "wait for approval"
                '''
                script {
                    input("do you want to procced?");
                }
                script 
                {
                    emailext subject: '${JOB_NAME} - ${BUILD_NUMBER} ', body: 'Job url : ${BUILD_URL}',  to: '${project_owner_team_email}'
                }
            }
        
        }
        stage('static-dockerfile-scan')
        {
            
            steps{
                sh '''
                echo "checov to scan the dockerfile"
                #docker run -t -v $(pwd):/output bridgecrew/checkov -f /output/Dockerfile -o json |  jq '.' > docker_result | exit 0
                #cat docker_result  | grep -B 2 FAILED
                #var=$(cat docker_result  | grep -ov -B 2 FAILED | wc -l)
                '''
            }
        
        }
        stage('docker-build')
        {
            
            steps{
                sh '''
                docker build -t rust-cart-app1:$app_version -f Dockerfile .
                echo "checov to scan the dockerfile"
                '''
            }
        
        }
        stage('docker-scan')
        {
            
            steps{
                sh '''
                echo "trivy scan"
                #install try
                #run try sh 'trivy image newbielinux1/rust-cart-app1'
                #condition to get HIGH | CRITICAL == 0
                #trivy image newbielinux1/rust-cart-app1 | grep HIGH
                #trivy image newbielinux1/rust-cart-app1 | grep CRITICAL
                '''
            }
        
        }
        stage('docker-push')
        {
            
            steps{
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker tag rust-cart-app1:$app_version newbielinux1/rust-cart-app1:$app_version
                docker push newbielinux1/rust-cart-app1:$app_version
                '''
            }
        
        }
                
    }
    post {
        success {
            script 
                {
                    emailext subject: '${JOB_NAME} - ${BUILD_NUMBER} Job Completed successfully ', body: 'Job url : ${BUILD_URL} ${currentBuild.fullDisplayName}',  to: '${project_owner_team_email}'
                }
        }
        failure {
            script 
                {
                    emailext subject: '${JOB_NAME} - ${BUILD_NUMBER} Job failed', body: 'Job url : ${BUILD_URL} ${currentBuild.fullDisplayName}',  to: '${project_owner_team_email}'
                }
        }
}
}