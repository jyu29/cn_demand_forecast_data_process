pipeline {

    agent any

    stages {

        stage("cluster provisioning") {

            steps {

            build job: "EMR-CREATE-PERSISTENT-CLUSTER",
                parameters: [
                    string(name: "nameOfCluster", value: "${BUILD_TAG}"),
                    string(name: "projectTag", value: "forecastinfra"),
                    string(name: "versionEMR", value: "emr-5.26.0"),
                    string(name: "instanceTypeMaster", value: "c5.2xlarge"),
                    string(name: "masterNodeDiskSize", value: "64"),
                    string(name: "nbrCoreOnDemand", value: "3"),
                    string(name: "nbrCoreSpot", value: "0"),
                    string(name: "instanceTypeCore", value: "m5.4xlarge"),
                    string(name: "coreNodeDiskSize", value: "64"),
                    string(name: "nbrTaskNode", value: "0"),
                    string(name: "instanceTypeTask", value: "c4.4xlarge"),
                    string(name: "taskNodeDiskSize", value: "64"),
                    string(name: "ldapUser", value: "aschwartz"),
                    string(name: "ldapGroup", value: "GR-DISCOVERY-ADM"),
                    string(name: "hdfsReplicationFactor", value: "3")
                    ]
            }

        }

        stage("spark app deployment and execution") {
            steps {
                wrap([$class: "BuildUser"]) {
                    sh('''
                    
                    export https_proxy="${https_proxy}"
                    
                    EMRName="forecast-emr-${BUILD_TAG}"
                    
                    cluster_id=$(aws emr list-clusters --active --output=json | jq '.Clusters[] | select(.Name=="'${EMRName}'") | .Id ' -r)
                    
                    instance_fleet_id=$(aws emr describe-cluster --cluster-id ${cluster_id} --output=json | jq '.Cluster.InstanceFleets[] | select(.InstanceFleetType=="MASTER") | .Id ' -r)
                    
                    master_ip=$(aws emr list-instances --cluster-id ${cluster_id} --output=json | jq '.Instances[] | select(.InstanceFleetId=="'${instance_fleet_id}'") | .PrivateIpAddress ' -r)
    
                    scp -r -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i /var/lib/jenkins/.ssh/${key_pem} ${WORKSPACE} hadoop@${master_ip}:/home/hadoop
    
                    ssh hadoop@${master_ip} -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i /var/lib/jenkins/.ssh/${key_pem} "export PYSPARK_PYTHON='/usr/bin/python3'; sudo chmod 755 /home/hadoop/${JOB_NAME}/main.sh; cd /home/hadoop/${JOB_NAME}; ./main.sh ${run_env}"
                    ''')
                }
            }
        }

        stage("delete cluster") {
            steps {
                build job: "EMR-DELETE-PERSISTENT-CLUSTER",
                    parameters: [
                        string(name: "nameOfCluster", value: "${BUILD_TAG}")
                   ]
            }
        }
    }
}
