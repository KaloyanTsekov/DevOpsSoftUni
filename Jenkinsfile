pipeline 
{
    agent any

    stages 
    {
        stage('Clone the project') 
        {
            steps 
            {
                sh '''
                    cd /projects
                    if [ -d /projects/DevOpsSoftUni ]; then 
                      cd /projects/DevOpsSoftUni
                      git pull https://github.com/KaloyanTsekov/DevOpsSoftUni.git 
                    else
                      git clone https://github.com/KaloyanTsekov/DevOpsSoftUni.git 
                    fi
                    '''
            }
        }
        stage('Build the APP image')
        {
            steps
            {
                sh 'cd /projects/DevOpsSoftUni/bgapp && docker image build -t img-app -f Dockerfile.web .'

            }
        }
        stage('Build the DB image')
        {
            steps
            {
                sh 'cd /projects/DevOpsSoftUni/bgapp && docker image build -t img-db -f Dockerfile.db .'
            }
        }
        stage('Create Network for the Containers')
        {
            steps
            {
                sh 'docker network ls | grep appnet || docker network create appnet'
            }
        }
        stage('Start the DB container')
        {
            steps
            {
                sh '''
                docker container rm -f co-img-app || true
                docker container run -d -p 9070:80 -v /projects/DevOpsSoftUni/bgapp/web:/var/www/html:ro --name co-img-app --net appnet img-app
                '''
            }
        }
        stage('Start the Web container')
        {
            steps
            {
                sh '''
                docker container rm -f db || true
                docker container run -d --name db --net appnet -e MYSQL_ROOT_PASSWORD=12345 img-db
                '''
            }
        }
    }
}
