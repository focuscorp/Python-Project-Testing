pipeline { 
   agent none 
   stages { 
       stage('Build') { 
         agent { 
              docker { 
                 image 'python:3-alpine' 
              } 
          } 
          steps { 
               sh 'python -m py_compile sources/*.py' 
               stash(name: 'compiled-results', includes: 'sources/*.py*') 
                              stash(name: 'setUpPy', includes: 'setup.py*') 
          } 
       } 
       stage('Unit Test') { 
         agent { 
              docker { 
                 image 'qnib/pytest:latest' 
              } 
         } 
         steps { 
              sh 'py.test --verbose --junit-xml test-reports/results.xml tests/*.py' 
         } 
         post { 
              always { 
                  junit 'test-reports/results.xml' 
              } 
         } 
       }
		stage('Packaging') { 
           agent any 
               environment { 
                   VOLUME = '$PWD/sources:/src' 
                   IMAGE = 'cdrx/pyinstaller-linux:python3' 
               } 
               steps { 
                   dir(path: env.BUILD_ID) { 
                       unstash(name: 'compiled-results') 
                       unstash(name: 'setUpPy') 
                       //https://docs.python.org/3/distutils/builtdist.html 
                       sh 'cd sources'  
                        sh 'ls -l'  
                        sh 'python3 setup.py bdist_dumb --format=zip' 
                    } 
               } 
               post { 
                   success { 
                        archiveArtifacts "${env.BUILD_ID}/dist/*" 
                   } 
               } 
        } 
    }
}

