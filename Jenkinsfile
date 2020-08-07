pipeline {
    agent any              //全局必须带有agent表明此pipeline执行节点
    tools {
        maven 'maven_home'
    }
    options {
        timeout(time: 1, unit: 'HOURS')
    }
    parameters {            //声明构建所需变量
        string(name: 'BRANCH', defaultValue: 'staging', description: '请输入git分支')
        choice(name: 'env', choices: ['uat', 'prod'], description: '请选择需要构建环境')
        gitParameter(branchFilter: 'origin/(.*)',
                     defaultValue: 'master', 
                     name: 'branch', 
                     type: 'PT_BRANCH_TAG', 
                     selectedValue: 'DEFAULT', 
                     sortMode: 'DESCENDING_SMART', 
                     description: '请选择发布的分支')
    }
    environment {
        select = "$params.projectNameChoose"
    }
    stages{

        stage("选择发布的项目") {
//            agent { label 'master && ceshi'}
            steps {
                script {
                    def branches = [:]
                    MAX_CONCURRENT = 4
                    latch = new java.util.concurrent.LinkedBlockingDeque(MAX_CONCURRENT)
                    for(int i=0; i<MAX_CONCURRENT; i++)
                        latch.offer("$i")
                    for (p_name in select.tokenize(',')){
                        def job_name = p_name.tokenize('"')[0]
                        echo "选择的项目为:" + job_name
                        echo "当前分支为:" + params.BRANCH
                        echo "当前环境为:" + params.env
                        branches[job_name] = {
                            def thing = null
                            waitUntil {
                                //获取一个资源
                                thing = latch.pollFirst();
                                return thing != null
                            }
                        }
                            try {
                                stage('当前执行工程:' + job_name){
                                    build(job: job_name, propagate: false, parameters: [gitParameter(name: 'BRANCH', value: params.BRANCH), string(name: 'env', value: params.env)])
//                          [$class: 'GitParameterValue', name: 'BRANCH', value: '${params.BRANCH}']
//                          parameters: [choice(name: 'env', value: '${params.env}')]     
                                }
                            }
                            finally {
                                latch.offer(thing)
                            }
                    }
                    timestamps {
                        parallel branches
                    }
                }
            }
        }

//        stage("选择构建节点和构建"){
//            agent { label 'master'}
//            steps{
//                echo "这是第一个步骤"
//                checkout([$class: 'GitSCM',
//                         branches: [[name: "${params.BRANCH}"]], 
//                         doGenerateSubmoduleConfigurations: false,
//                         extensions: [], 
//                         gitTool: 'Default', 
//                         submoduleCfg: [], 
//                         userRemoteConfigs: [[url: 'https://github.com/yeji-python/pipeline_test.git', credentialsId: '69d10d50-d671-415c-9b38-e787ed3184fd']]
//                       ])                
//            }
//       }
//        stage("maven打包和发布"){
//            agent { label 'master'}
//            steps{
//               sh "mvn clean package toolkit:deploy -DskipTests=true -Dedas_config=${params.env}-edas-config.yaml"  
//            }
//        }
    }
}
