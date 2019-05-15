def dockernode;
def dockerip;
def dockip;

node('linux') {
    
    stage('Create Test Stack') {

            git credentialsId: 'Github-Token-ID', url: 'https://github.com/UST-SEIS665/final-seis665-01-spring2019-mamry.git'
    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS-Password-ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    
                    sh 'aws cloudformation create-stack --stack-name final-test --region us-east-1 --template-body file://docker-single-server.json --parameters ParameterKey=KeyName,ParameterValue=midKey ParameterKey=YourIp,ParameterValue=$(curl ifconfig.me)/32'
                    sh 'aws cloudformation wait stack-create-complete --stack-name final-test --region us-east-1'
                    sh 'aws cloudformation describe-stacks --stack-name final-test --region us-east-1'
                    returnStdout: true

                    sshagent(['ssh-agent-ID']){
                        //dockernode=$(aws cloudformation describe-stacks --stack-name final-test --region us-east-1 --query Stacks[].Outputs[].[OutputValue] --output text)
                        
                        //dockerip=$(aws ec2 describe-instances --region us-east-1 --filters "Name=image-id,Values=ami-043218c94b0cb8d43" --query "Reservations[*].Instances[*].PublicIpAddress")

                        sh '''
                        aws ec2 describe-instances --region us-east-1 --filters "Name=image-id,Values=ami-043218c94b0cb8d43" --query "Reservations[*].Instances[*].PublicIpAddress" > dockip
                        cat dockip
                        cat dockip | tr -d '[]",[:space:]' > dockerip
                        cat dockerip
                        '''
                        //ssh -o StrictHostKeyChecking=no ubuntu@$dockernode uptime
                        sshagent(['ssh-agent-ID']) {
                         sh 'ssh -o StrictHostKeyChecking=no ubuntu@$(cat dockerip) uptime'
                        }

                    
                    }
            }
    }
    
     stage('Deploy Redis Standalone') {
         
         sshagent(['ssh-agent-ID']) {
           
             sh 'ssh ubuntu@$(cat dockerip) \' docker run -d redis:latest -p 6379:6379 \''
             //sh 'ssh ubuntu@$(cat dockerip)  docker run --name myredis -d redis:latest -p 6379:6379'


            //sh '''
            //ssh -o StrictHostKeyChecking=no ubuntu@$dockerip -d --name redis1 -p 6379:6379 redis:latest'
            //'''
             
         }
     }
    
    stage('Test Redis Standalone') {
        
        sshagent(['ssh-agent-ID']) {
           
            //ssh -o StrictHostKeyChecking=no ubuntu@$(cat dockerip) \'redis-cli set hello world\'
           //ssh -o StrictHostKeyChecking=no ubuntu@$(cat dockerip) \'redis-cli get hello\'
       
//ssh -o StrictHostKeyChecking=no ubuntu@${dockerip} redis-cli set hello world
//ssh -o StrictHostKeyChecking=no ubuntu@${dockerip} redis-cli get hello world
            sh '''
               ssh -o StrictHostKeyChecking=no ubuntu@$(cat dockerip) redis-cli set hello world
               ssh -o StrictHostKeyChecking=no ubuntu@$(cat dockerip) redis-cli get hello world
             '''
        }
    }
    
   /* stage('Delete Test Stack') {
            withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding', 
                accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                credentialsId: '5f6722b0-b29e-4bfc-86c2-10b9ca813672', 
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) 
                {  
            sh 'aws cloudformation delete-stack --stack-name final-test --region us-east-1'
            sh 'aws cloudformation wait stack-delete-complete --stack-name final-test --region us-east-1'
        }
    }*/ 
}
