pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        EB_APPLICATION_NAME = 'HealthAxis'
        EB_ENVIRONMENT_NAME = 'HealthAxis-dev'
        S3_BUCKET = 'healthaxis-jenkins-017118230156-ap-south-1-an '
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Angular') {
            steps {
                dir('HealthAxis3.AngularUI') {
                    bat 'npm ci'
                    bat 'npm run build'
                }
            }
        }

        stage('Publish Blazor') {
            steps {
                bat 'dotnet publish HealthAxis3.BlazorUI\\HealthAxis3.BlazorUI.csproj -c Release -o blazor-publish-temp'
            }
        }

        stage('Copy Blazor into API wwwroot') {
            steps {
                bat 'if not exist HealthAxis3.Api\\wwwroot\\blazor mkdir HealthAxis3.Api\\wwwroot\\blazor'
                bat 'xcopy /E /Y /I blazor-publish-temp\\wwwroot\\* HealthAxis3.Api\\wwwroot\\blazor\\'
            }
        }

        stage('Publish API') {
            steps {
                bat 'dotnet publish HealthAxis3.Api\\HealthAxis3.Api.csproj -c Release -o publish'
            }
        }

        stage('Zip published output') {
    steps {
        dir('publish') {
            bat 'jar -cMf ../deploy-package.zip .'
        }
    }
}

        stage('Upload to S3 and Deploy to EB') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Melsy']]) {
                    bat "aws s3 cp deploy-package.zip s3://%S3_BUCKET%/deploy-package-%BUILD_NUMBER%.zip --region %AWS_REGION%"

                    bat "aws elasticbeanstalk create-application-version --application-name %EB_APPLICATION_NAME% --version-label v-%BUILD_NUMBER% --source-bundle S3Bucket=%S3_BUCKET%,S3Key=deploy-package-%BUILD_NUMBER%.zip --region %AWS_REGION%"

                    bat "aws elasticbeanstalk update-environment --environment-name %EB_ENVIRONMENT_NAME% --version-label v-%BUILD_NUMBER% --region %AWS_REGION%"
                }
            }
        }
    }
}