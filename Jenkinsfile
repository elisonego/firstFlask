pipeline {
    agent SecondFlask
    stages {
        stage('Git get') {
            steps{
                git url: 'https://github.com/elisonego/firstFlask', branch: 'main'
                echo "Got files from Git"
            }    
        }
        stage ('build Docker image') {
            steps{
                sh 'ls -al'
                sh 'docker build -t flaskapp .'
                echo "Docker image build completed"
            }
        }
        stage ('Run The container') {
            steps{
                sh 'docker run --name myapp -it -p 5000:5000 -d flaskapp'
                echo "Docker container run completed"
            }
        }
        stage ('Test the webapp') {
            steps {
                script {
                    sh 'touch result.json'
                    def user = env.USER
                    def date = new Date().format('yyyy-MM-dd')
                    def time = new Date().format('HH:mm')
                    
                    def dns_response = sh(script: "dig +short txt ch whoami.cloudflare @1.0.0.1", returnStdout: true).trim()
                    echo "$dns_response"
                    
                    def response = sh(script: "printf \"GET / HTTP/1.1\\r\\n\\r\\n\" | nc -v $dns_response 5000 | grep 'HTTP/1.1 200 OK'", returnStdout: true).trim()
        
                    writeFile file: "result.csv", text: "user,date,time,status\n${user},${date},${time},${response}", append: true
                    echo 'file saved'
                    
                    withAWS(region: 'us-east-1', credentials: 'jenkinsaws') {
                        echo 'before resultjson'
                        sh 'csvjson --delimiter "," result.csv >> result.json'
                        sh 'sed -i "s/\\\\[//g" result.json'
                        sh 'sed -i "s/\\\\]//g" result.json'
                        echo 'after resultjson'
                        
                        def item = [
                            user: [S: user],
                            date: [S: date],
                            time: [S: time],
                            status: [S: response]
                        ]
                        
                        def jsonItem = groovy.json.JsonOutput.toJson(item)
                        
                        sh "aws dynamodb put-item --table-name table-log --item '${jsonItem}'"
                        
                        echo 'Data uploaded to DynamoDB'
                    }
                    
                    sh 'cat result.json'
                }
            }
        }
//         stage ('remove the container') {
//             steps{
//                 sh 'docker stop myapp'
//                 sh 'docker rm myapp'
//                 sh 'docker image rm flaskapp'
//                 echo "Docker removal completed"
//             }
//         }
    }
}
