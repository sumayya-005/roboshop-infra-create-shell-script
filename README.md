#!/bin/bash
######change these Values###
ZONE_ID="Z06198638E0IJOLZ2M33"
SG_NAME="allow-all"


CREATE_ec2(){
 PRIVATE_ip=$(aws ec2 run-instances\
         --image-id ${AMI_id} \
          --instance-type  t3.micro \
          --tag-specifications"ResourceType=instance,Tags=[{Key=Name,Value=${COMPONENT}}]") \
          --security-group-ids ${SGID} \
           |jq '.Instances[].PrivateIpAddress' | sed -e 's/"//g')
  
sed -e "s/IPADDRESS/${PRIVATE_IP}/"-e "s/COMPONENT/${COMPONENT}"/ route53.json >/tmp/record.json
aws route53 change-resource-record-sets --hosted-zone-id ${ZONE_ID} --change-batch file:///tmp/record.json | jq

}

##Main Program
  AMI_ID=$(aws ec2 describe-images --filters "Name=name,Values=Centos-8-Devops-Practice" | jq '.Images[].ImageId' | sed -e 's/"//g')
if [ -z "${AMI_ID}" ]; then
   echo "AmI_ID not found"
    exit 1
fi

  SG_ID=$(aws ec2 describe-security-groups --filters Name=group-name,Values=${SG_Name} |jq '.SecurityGroups[].GroupId' | sed -e 's/"//g')
  if [ -z "${SGID}" ]; then
    echo "Given Security Group Does not exit"
     exit 1
  fi
  
for component in catalogue cart user shipping payment frontend mongodb mysql rabbitmq redis dispatch; do 
    COMPONENT="${component}"
     create_ec2
done