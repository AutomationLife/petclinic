node ('DockerIO') {
    stage 'Checkout'
    //git "https://github.com/alekya123/petclinic.git"
    checkout scm
    echo "CHeckout Code"

    stage 'Build application war file'
    // Build petclinic in a Maven3+JDK8 Docker container
    docker.image('maven:3-jdk-8').inside('-v /.m2:/root/.m2') {
        sh 'mvn -B package -DskipTests'
    }

    stage 'Build application Docker image'
    def appImg = docker.build("petclinic")

    stage 'Push to GCR'
       sh ''' ibmcloud login --apikey pukDLNLc1Csk5RXHJs1WpOJKYE-V0aK6U2lpvv4PLjB6 &&
        ibmcloud cr login &&
        docker tag petclinic us.icr.io/liberty_test/petclinic:latest &&
        docker push us.icr.io/liberty_test/petclinic:latest '''
   

    stage ('UCD Deploy') {
	           script {		
		    echo "started deploying in UCD in ${env.WORKSPACE}"
		    //echo '${COMPONENT_NAME}'
		    //echo "COMPONENT_NAME = 'Multi-Cloud'
		    sh 'echo "${BUILD_NUMBER}">tag ; cat tag'
		    step([  $class: 'UCDeployPublisher',
                    siteName: 'IBM GBS UCD',
                    component: [
                    $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
			    componentName: 'Multi-Cloud',
		    delivery: [
                    	    $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
                   	    pushVersion: "${BUILD_NUMBER}_pet",
			    baseDir: "${env.WORKSPACE}",
			    fileIncludePatterns: 'iksdeploy.yml'
                             ]
                    ],
                    deploy: [
                 $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeployHelper$DeployBlock',
                 deployApp: 'Multicloud-Petclinc',
                 deployEnv: 'IKS',
                 deployProc: "deploy-Multi-Cloud",
                 deployVersions: "Multi-Cloud:${BUILD_NUMBER}_pet",
                 deployOnlyChanged: false
                         ]
                           ])
			   }
	}
	

    // ... Do some tests on deployed application web UI
}
