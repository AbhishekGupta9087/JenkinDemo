import groovy.transform.Field

@Field
def pipelineLogLevel
@Field
def lastStageName

def masterInstanceIPs = []
def dataInstanceIPs = []
def coorInstanceIPs = []
def masterProcessId
def dataProcessId
def coorProcessId
def master_namespace_variation = ''
def data_namespace_variation = ''
def coor_namespace_variation = ''
def oldClusterName
def activity


def printCompleteStackTrace(Exception e) {
    def stackTraceStr = ""
    def stackTrace = e.getStackTrace()
    int stackSize = stackTrace.size()
    for ( int i =0;i<stackSize;i++){
        if(i==stackSize-1){
            stackTraceStr += stackTrace[i].toString()
        }else {
            stackTraceStr += stackTrace[i].toString() + '\n'
        }
    }
    return stackTraceStr
}

def logError(Exception e,String stageName) {
    println 'inside logError function'
    try {
        long tid = Thread.currentThread().getId()
        def buildNum = env.BUILD_NUMBER
        def clusterName = params.'namespace'
        if (params.'namespace_variation' != '') {
            clusterName = clusterName +'-'+params.'namespace_variation'
        }
        def requestor = params.'requestor'
        def className = e.getClass().getName()
        def logType = "ERROR"
        def expMessage = e.getMessage()
        def pste = printCompleteStackTrace(e)
        def ste = expMessage+'\n'+pste
        Thread.sleep(1000)
        def logStr = String.format("[%d] [%s] Build_Number:%s Requestor:%s Cluster_Name:%s Stage_Name:%s - [%s] %s"
        , tid,logType,buildNum,requestor,clusterName,stageName,className,ste)
        def cmd = sprintf('echo "%s" >> /var/log/jenkinslog/errorlogs',logStr)
        def ret = sh(script: cmd , returnStdout: true) 
        //sh cmd
    }catch(exp){
        println 'printing exception'
        println exp
    }    
}

def logMessage(String message,String stageName,int logLevel) {
    println ' log message pipeline number :  '+pipelineLogLevel
    if (pipelineLogLevel<logLevel){
        return 
    }
    println 'inside logInfo function'
    try {
        long tid = Thread.currentThread().getId()
        def buildNum = env.BUILD_NUMBER
        def clusterName = params.'namespace'
        if (params.'namespace_variation' != '') {
            clusterName = clusterName +'-'+params.'namespace_variation'
        }
        def requestor = params.'requestor'
        def className = this.getClass().getName()
        def logType 
        if(logLevel==1){
            logType = "INFO"
        }else if(logLevel==2){
            logType = "DEBUG"
        }
        def logStr = String.format("[%d] [%s] Build_Number:%s Requestor:%s Cluster_Name:%s Stage_Name:%s - [%s] %s"
        , tid,logType,buildNum,requestor,clusterName,stageName,className,message)
        def cmd = sprintf('echo "%s" >> /var/log/jenkinslog/buildlogs',logStr)
        println 'before logInfo cmd'
        def ret = sh(script: cmd , returnStdout: true) 
        //sh cmd
        println 'after logInfo cmd'
    }catch(exp){
        println 'printing exception'
        println exp
    }    
}

def slackNotification(String message ){

    def requestor = params.'requestor'
    def userId = slackUserIdFromEmail(params.'requestor')
    def response = httpRequest url: "https://slack.com/api/users.info?user="+userId,
                            httpMode: 'GET',
                            customHeaders: [[name: "Authorization", value: "Bearer ${env.slack_token}"]]
    
    def fullName = (new groovy.json.JsonSlurper().parseText(response.content)).user.profile.real_name
    
    blocks= [
        [
           "type": "section",
          "text": [
               "type": "mrkdwn",
               "text": "Dear $fullName, \n :slack: : <@$userId> \n :email: : $requestor \n\n  $message."
              ]
        ],
        [
           "type": "divider"
        ],
        [
           "type": "section",
           "text": [
                "type": "mrkdwn",
                "text": "*Summary*\n"+ params.'summary'
                ]  
        ],
        [
            "type": "divider"
        ],
        [
            "type": "section",
            "text": [
                "type": "mrkdwn",
                "text": "Thanks for using ES-platform automation product :smile_cat: \n*cc:* <!subteam^S02G51DB1AS|search-platform-dev>"
                ]
        ]
   ]
   println "blocks class- "+blocks.getClass()
   slackSend channel: '#wg-es-platform', blocks: blocks, iconEmoji: ':laptop:', username: 'ES Platform'
}

def createAgoraInstance(def suffix, def namespace_variation){
    def extraDisk = ''
    def instance_count = params.'Number of Master Nodes'
    def instance_type = params.'master_instance_type'
    def default_extra_disk_size = "20"
    def default_extra_disk_type = "pd-standard"
    def master_extra_disk = params.'Master Extra Disk Size'
 
 
    //Setup Extra Disk Data
    if (suffix == 'data'){

        //get from input user
        def data_extra_disk_size = params.'Data Extra Disk Size'
        def data_extra_disk_type = params.'Data Extra Disk Type'

        //validation input data disk size, if empty set default size
        if (data_extra_disk_size == "") {
            data_extra_disk_size = default_extra_disk_size
        }

         //validation input data disk size, if empty set default type
        if (data_extra_disk_type == "") {
            data_extra_disk_type = default_extra_disk_type
        }
        
        extraDisk = ',{ "mount_path": "data", "size": ' +  data_extra_disk_size + ', "type": "' +  data_extra_disk_type + '"}'
        instance_type = params.'data_instance_type'
        instance_count = params.'Number of Data Nodes'
    }else if (suffix == 'coor'){

        //get from input user
        def coor_extra_disk_size = params.'Coor Extra Disk Size'
        def coor_extra_disk_type = params.'Coor Extra Disk Type'

        //validation input data disk size, if empty set default size
        if (coor_extra_disk_size == "") {
            coor_extra_disk_size = default_extra_disk_size
        }

         //validation input data disk size, if empty set default type
        if (coor_extra_disk_type == "") {
            coor_extra_disk_type = default_extra_disk_type
        }
        
        extraDisk = ',{ "mount_path": "data", "size": ' +  coor_extra_disk_size + ', "type": "' +  coor_extra_disk_type + '"}'
        instance_type = params.'coor_instance_type'
        instance_count = params.'Number of Coor Nodes'
    }else if (suffix == 'master' && master_extra_disk){

        //get from input user
        def master_extra_disk_type = params.'Master Extra Disk Type'
        def master_extra_disk_size = params.'Master Extra Disk Size'

        //validation input master disk size, if empty set default size
        if (master_extra_disk_size == "") {
            master_extra_disk_size = default_extra_disk_size
        }

        //validation input master disk type, if empty set default type
        if (master_extra_disk_type == ""){
            master_extra_disk_type = default_extra_disk_type
        }
        
        extraDisk = ',{ "mount_path": "data", "size": ' +  master_extra_disk_size + ', "type": "' +  master_extra_disk_type + '"}'
    }
    
    retry(3){
        println "extradisk"+ extraDisk
        println '{"metadata": {"directorate":"'+"${params.directorate}"+'", "tech_directorate":"'+"${params.tech_directorate}"+'", "cost_center":"'+"${params.cost_center}"+'", "vpe_approver":"'+"${params.vpe_approver}"+'", "em_approver":"'+"${params.em_approver}"+'", "singular_leader":"'+"${params.singular_leader}"+'", "requestor":"'+"${params.requestor}"+'", "squad":"'+"${params.squad}"+'", "tribe":"'+"${params.tribe}"+'", "summary":"'+"${params.summary}"+'", "workspace": "gcp-infra-staging-asia-southeast-1-staging", "is_preview": false, "issue":"'+"${params.issue}"+'", "source":"'+"${params.source}"+'", "webhook_provider": "create_general_instance" }, "request": { "datacenter":"'+"${params.datacenter}"+'", "resource_type": "general", "namespace":"sp-es-'+"${params.namespace}"+'", "namespace_variation":"'+ namespace_variation +'", "general_instance_spec": { "instance_count": ' + instance_count +', "image":"'+"${params.image}"+'", "instance_type":"'+instance_type+'", "disk_size": '+"${params.disk_size}"+', "service_type": "app", "additional_disk": [ { "mount_path": "var/log", "size": 40, "type": "pd-standard" } ' + extraDisk +' ], "optional_vars_userdata": [ "export A=b" ], "optional_vars_suffix_userdata": [ "export A=b" ], "additional_tags": { "a": "b",  "c": "d" }, "gcp_project_id": "'+"${params.gcp_project_id}"+'" } } }'
        try{
            def response = httpRequest httpMode: 'POST',
                                      contentType: 'APPLICATION_JSON',
                                      url: 'http://172.18.16.219:9000/v1/gcp/create/instance',
                                      requestBody: '{"metadata": {"directorate":"'+"${params.directorate}"+'", "tech_directorate":"'+"${params.tech_directorate}"+'", "cost_center":"'+"${params.cost_center}"+'", "vpe_approver":"'+"${params.vpe_approver}"+'", "em_approver":"'+"${params.em_approver}"+'", "singular_leader":"'+"${params.singular_leader}"+'", "requestor":"'+"${params.requestor}"+'", "squad":"'+"${params.squad}"+'", "tribe":"'+"${params.tribe}"+'", "summary":"'+"${params.summary}"+'", "workspace": "gcp-infra-staging-asia-southeast-1-staging", "is_preview": false, "issue":"'+"${params.issue}"+'", "source":"'+"${params.source}"+'", "webhook_provider": "create_general_instance" }, "request": { "datacenter":"'+"${params.datacenter}"+'", "resource_type": "general", "namespace":"sp-es-'+"${params.namespace}"+'", "namespace_variation":"'+ namespace_variation +'", "general_instance_spec": { "instance_count": '+instance_count +', "image":"'+"${params.image}"+'", "instance_type":"'+instance_type+'", "disk_size": '+"${params.disk_size}"+', "service_type": "app", "additional_disk": [ { "mount_path": "var/log", "size": 40, "type": "pd-standard" } ' + extraDisk +' ], "optional_vars_userdata": [ "export A=b" ], "optional_vars_suffix_userdata": [ "export A=b" ], "additional_tags": { "a": "b",  "c": "d" }, "gcp_project_id": "'+"${params.gcp_project_id}"+'" } } }',
                                      customHeaders: [[name: 'agora-webhook-token', value: env.AGORA_TOKEN]]
            println response.content
                                        
            def jsonResponse = new groovy.json.JsonSlurper().parseText(response.content)
            def id = jsonResponse.id
            println id
            return id
        } catch(Exception e){
            echo "instance creation exception" + e
            error 'instance creation exception'
        }
    }
    
}

//check healty instance consul hostname
def checkHealtyConsulHostName(def hostName){
    def response = httpRequest url: "https://consul-gcp-staging.tokopedia.net/v1/health/node/${hostName}"
  
    //checking for response API header status
    if ( response.status != 200 ){
         error("Error : $response.status")
    }
    
    //parsing json response API
    def jsonResponse = new groovy.json.JsonSlurper().parseText(response.content)
    
    //loop the data response
    i = 1
    jsonResponse.each { key, value ->
        // if status passing and name == Serf Health Status show the output data
        if (key.Status == "passing" && key.Name == "Serf Health Status") {
            echo "The service is healthy : "
            println "Service - " + i
            println "Node : $key.Node"
            println "CheckID : $key.CheckID"
            println "Name : $key.Name"
            println "Status : $key.Status"
            println "Notes : $key.Notes"
            println "Output : $key.Output"
            println "ServiceID : $key.ServiceID"
            println "ServiceName : $key.ServiceName"
            println "ServiceTags : $key.ServiceTags"
            println "Type : $key.Type"
            println "\n\n"
        }else{
            echo "The service is not healthy : "
            echo "Node : $key.Node"
        }
    }
}

pipeline {
    agent any
    environment{
        AGORA_TOKEN = credentials('agora_token')
        slack_token = credentials('slack_notification')
    }
    
    stages {
        stage('log') {
            steps {
                echo "log in ES: Received request to spawn Cluster"
                
                script{
                    if(params.'namespace_variation' != '') {
                        master_namespace_variation = params.'namespace_variation' + '-master'
                        data_namespace_variation = params.'namespace_variation' + '-data'
                        coor_namespace_variation = params.'namespace_variation' + '-coor'
                    }else {
                        master_namespace_variation = 'master'
                        data_namespace_variation = 'data'
                        coor_namespace_variation = 'coor'
                    }
                    oldClusterName = params.'old_cluster_name'
                    activity = params.'activity'
                    
                    def pipelineLogString = params.'Log Level'
                    if ( pipelineLogString == 'INFO') {
                        pipelineLogLevel = 1
                    }else if( pipelineLogString == 'DEBUG'){
                         pipelineLogLevel = 2
                    }
                    
                    println 'pipelineLogLevel :  '+pipelineLogLevel

                }
            }
        }
        stage('Verify Hostname') {
            steps {
                echo "Verify Hostname"
                
                script{
                    def master_node_count = params.'Number of Master Nodes'
                    def data_node_count = params.'Number of Data Nodes' 
                    def coor_node_count = params.'Number of Coor Nodes'
                    def clusterName = params.'namespace'

                    if (master_node_count > 0) {
                        masterHostname = clusterName +'-'+ master_namespace_variation
                        def checkHostname = checkHealtyConsulHostName(masterHostname)
                        if (checkHostname == "The service is healthy") {
                            echo "Master Node was created, please check another namespace"
                            error 'Master Node was created, please check another namespace'
                        }
                    }

                    if (data_node_count > 0) {
                        dataHostname = clusterName +'-'+ data_namespace_variation
                        def checkHostname = checkHealtyConsulHostName(dataHostname)
                        if (checkHostname == "The service is healthy") {
                            echo "Data Node was created, please check another namespace"
                            error 'Data Node was created, please check another namespace'
                        }
                    }

                    if (coor_node_count > 0) {
                        coorHostname = clusterName +'-'+ coor_namespace_variation
                        def checkHostname = checkHealtyConsulHostName(coorHostname)
                        if (checkHostname == "The service is healthy") {
                            echo "Coor Node was created, please check another namespace"
                            error 'Coor Node was created, please check another namespace'
                        }
                    }
                }
            }
        }
}
