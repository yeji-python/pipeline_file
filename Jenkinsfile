pipeline {
    agent none               //全局必须带有agent表明此pipeline执行节点
    tools {
        maven 'maven_home'
    }
    options {
        timeout(time: 1, unit: 'HOURS')
    }
    parameters {            //声明构建所需变量
        choice(name: 'env', choices: ['uat', 'prod'], description: '请选择需要构建环境')
        gitParameter(branchFilter: 'origin/(.*)',
                     defaultValue: 'master', 
                     name: 'BRANCH', 
                     type: 'PT_BRANCH_TAG', 
                     selectedValue: 'DEFAULT', 
                     sortMode: 'DESCENDING_SMART', 
                     description: '请选择发布的分支')
    }
    stages{
        stage("选择构建节点和构建"){
            agent { label 'master'}
            steps{
                echo "这是第一个步骤"
                checkout([$class: 'GitSCM',
                         branches: [[name: "${params.BRANCH}"]], 
                         doGenerateSubmoduleConfigurations: false,
                         extensions: [], 
                         gitTool: 'Default', 
                         submoduleCfg: [], 
                         userRemoteConfigs: [[url: 'https://github.com/yeji-python/pipeline_test.git', credentialsId: '69d10d50-d671-415c-9b38-e787ed3184fd']]
                       ])                
            }
        }
        stage("maven打包和发布"){
            agent { label 'master'}
            steps{
                sh "mvn clean package toolkit:deploy -DskipTests=true -Dedas_config=${params.env}-edas-config.yaml"  
            }
        }
    }
}
