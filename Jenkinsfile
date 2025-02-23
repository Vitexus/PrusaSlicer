#!groovy

// Docker images generated by  https://github.com/VitexSoftware/BuildImages


pipeline {
    agent none

    options {
        ansiColor('xterm')
        copyArtifactPermission('*');
	disableConcurrentBuilds()
    }

    environment {
        RED      = '\\e[31m'
        GREEN    = '\\e[32m'
        ENDCOLOR = '\\e[0m'
    }    
    
    stages {

/*
        stage('debian-stable') {
            agent {
                docker { image 'vitexsoftware/debian:stable' }
            }
            steps {
                dir('build/debian/package') {
                    checkout scm
		            buildPackage()
		            installPackages()
                }
                stash includes: 'dist/**', name: 'dist-buster'
            }
            post {
                success {
                    archiveArtifacts 'dist/debian/'
                    copyArtifact()
                }
            }


            
        }

        stage('debian-testing') {
            agent {
                docker { image 'vitexsoftware/debian:testing' }
            }
            steps {
                dir('build/debian/package') {
                    checkout scm
		            buildPackage()
		            installPackages()
                }
                stash includes: 'dist/**', name: 'dist-bullseye'
            }
            post {
                success {
                    archiveArtifacts 'dist/debian/'
                    copyArtifact()
                }
            }
        }

        stage('ubuntu-focal') {
            agent {
                docker { image 'vitexsoftware/ubuntu:stable' }
            }
            steps {
                dir('build/debian/package') {
                    checkout scm
		            buildPackage()
		            installPackages()
                }
                stash includes: 'dist/**', name: 'dist-focal'
            }
            post {
                success {
                    archiveArtifacts 'dist/debian/'
                    copyArtifact()
                }
            }
        }
*/

        stage('ubuntu-hirsute') {
            agent {
                docker { image 'vitexsoftware/ubuntu:testing' }
            }
            steps {
                dir('build/debian/package') {
                    checkout scm
		            buildPackage()
		            installPackages()
                }
                stash includes: 'dist/**', name: 'dist-hirsute'
            }
            post {
                success {
                    archiveArtifacts 'dist/debian/'
                    copyArtifact()
                }
            }
        }

    }
}

def copyArtifact(){
    step ([$class: 'CopyArtifact',
        projectName: '${JOB_NAME}',
        filter: "**/*.deb",
        target: '/var/tmp/deb',
        flatten: true,
        selector: specific('${BUILD_NUMBER}')
    ]);
}

def buildPackage() {

    def DIST = sh (
	script: 'lsb_release -sc',
        returnStdout: true
    ).trim()

    def DISTRO = sh (
	script: 'lsb_release -sd',
        returnStdout: true
    ).trim()


    def SOURCE = sh (
	script: 'dpkg-parsechangelog --show-field Source',
        returnStdout: true
    ).trim()

    def VERSION = sh (
	script: 'dpkg-parsechangelog --show-field Version',
        returnStdout: true
    ).trim()

    ansiColor('vga') {
      echo '\033[42m\033[90mBuild debian package ' + SOURCE + ' v' + VERSION  + ' for ' + DISTRO  + '\033[0m'
    }

    def VER = VERSION + '~' + DIST + '~' + env.BUILD_NUMBER 

//Buster problem: Can't continue: dpkg-parsechangelog is not new enough(needs to be at least 1.17.0)
//
//    debianPbuilder additionalBuildResults: '', 
//	    components: '', 
//	    distribution: DIST, 
//	    keyring: '', 
//	    mirrorSite: 'http://deb.debian.org/debian/', 
//	    pristineTarName: ''
    sh 'dch -b -v ' + VER  + ' "' + env.BUILD_TAG  + '"'
    sh 'sudo apt-get update'
    sh 'debuild-pbuilder  -i -us -uc -b'
    sh 'mkdir -p $WORKSPACE/dist/debian/ ; rm -rf $WORKSPACE/dist/debian/* ; mv ../' + SOURCE + '*_' + VER + '_*.deb ../' + SOURCE + '*_' + VER + '_*.changes ../' + SOURCE + '*_' + VER + '_*.build $WORKSPACE/dist/debian/'
}

def installPackages() {
    sh 'cd $WORKSPACE/dist/debian/ ; dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz; cd $WORKSPACE'
    sh 'echo "deb [trusted=yes] file:///$WORKSPACE/dist/debian/ ./" | sudo tee /etc/apt/sources.list.d/local.list'
    sh 'sudo apt-get update'
    sh 'echo "${GREEN} INSTALATION ${ENDCOLOR}"'
    sh 'IFS="\n\b"; for package in  `ls $WORKSPACE/dist/debian/ | grep .deb | awk -F_ \'{print \$1}\'` ; do  echo -e "${GREEN} installing ${package} on `lsb_release -sc` ${ENDCOLOR} " ; sudo  DEBIAN_FRONTEND=noninteractive DEBCONF_DEBUG=developer apt-get -y install $package ; done;'
}
