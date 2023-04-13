def doSum(def a,def b){
    return a+b;
}
def doSub(def a,def b){
    return b-a;
}
def doMul(def a,def b){
    return a*b;
}
def doDiv(def a,def b){
    return b/a;
}

def a = 10
def b = 20

pipeline {
    agent any

    stages {
        stage('Add Stage') {
            steps {
                script{
                    println "add number"
                    println doSum(a,b);
                }
            }
        }
        stage('Multiply Stage') {
            steps {
                script{
                    println "multiple number"
                    println doMul(a,b);
                }
            }
        }
        stage('subtract Stage') {
            steps {
                script{
                    println "substract number"
                    println doSub(a,b);
                }
            }
        }
    }
}
