# metirials
'''
import boto3
def lambda_handler(event, context):

    #-------------------- Debug ---------------------------
    print( 'Hello  {}'.format(event))
    
    # -------------- Parameters --------------
    # 
    loadBalancersList = {
      "arn:aws:elasticloadbalancing:ap-southeast-1:027065296145:targetgroup/bluegreen-tg/b93ed9788fb7b344": "arn:aws:elasticloadbalancing:ap-southeast-1:027065296145:targetgroup/bluegreen-tg-internal/cb7a4a08b50a099e",
      "ARNTargetgroupExternal1": "ARNTargetgroupInternal1",
      "ARNTargetgroupExternal2": "ARNTargetgroupInternal2"
    }
    orignTargetGroupARN = event['detail']['requestParameters']['targetGroupArn']
    print(orignTargetGroupARN)
    if orignTargetGroupARN in loadBalancersList:
      newTargetGroupARN   = loadBalancersList[orignTargetGroupARN]
    else:
      print("The loadbalancer ARN did not exist in the dictionary, exiting lambda")
      exit()

    regionName = event['detail']['awsRegion']
    #debug print(regionName)
    #debug print(orignTargetGroupARN)
    #debug print(newTargetGroupARN)
    listOfEc2IdNew = []
    listOfEc2IdOrig = []   
    positiveStates = ["initial","healthy","unhealthy","unavailable","unused"]
    
    # ------------ Object's connection clients --------------
    elbClient = boto3.client('elbv2', region_name = regionName)
    
    # Acquiring json objects
    listOfTargetsOriginalJson = elbClient.describe_target_health(TargetGroupArn = orignTargetGroupARN )
    listOfTargetsNewJson  = elbClient.describe_target_health(TargetGroupArn = newTargetGroupARN  )
    
    # Creating the new EC2 target group list (listOfEc2IdNew)
    for target in listOfTargetsNewJson['TargetHealthDescriptions']:
        id = target['Target']['Id']
        state = target['TargetHealth']['State']
        if state in positiveStates:
          listOfEc2IdNew.append(id)
          
    #debug print(listOfTargetsNewJson)
    
    # Registering new targets in the new target group & creating the original target group EC2 list (listOfEc2IdOrig)
    for target in listOfTargetsOriginalJson['TargetHealthDescriptions']:
        id = target['Target']['Id']
        state = target['TargetHealth']['State']
        if state in positiveStates:
          listOfEc2IdOrig.append(id)
          if id not in listOfEc2IdNew:
            #print("Yes," + id + " found in List : " )
            port = target['Target']['Port']
            # Registering new targets in the new target group
            response = elbClient.register_targets(
                TargetGroupArn = newTargetGroupARN,
                Targets=[
                {
                  'Id': id ,
                  'Port': port
                },
              ]
            )
        
    #debugprint("----------------------------------")
    #debugprint("Orig" , listOfEc2IdOrig)
    #debugprint("New"  , listOfEc2IdNew)
    
    # Re-evaluating the new list
    listOfTargetsNewJson = elbClient.describe_target_health(TargetGroupArn = newTargetGroupARN)
    listOfEc2IdNew = []   
     
    for target in listOfTargetsNewJson['TargetHealthDescriptions']:
        id = target['Target']['Id']
        state = target['TargetHealth']['State']
        if state in positiveStates:
          listOfEc2IdNew.append(id)
          #print("Yes," + id + " found in List " )
      
          
    # Dissection between two lists to find the the old targets that needs to be deleted
    listOfEc2IdOrig = set(listOfEc2IdOrig)
    listOfEc2IdNew  = set(listOfEc2IdNew)
    listOfEc2ToDeRegisterTargets = listOfEc2IdNew.difference(listOfEc2IdOrig)
    
    #debugprint("############################################")
    #debugprint("Orig" , listOfEc2IdOrig)
    #debugprint("New"  , listOfEc2IdNew)
    #debugprint("Dissection" , listOfEc2ToDeRegisterTargets)

    
    for id in listOfEc2ToDeRegisterTargets:
      response = elbClient.deregister_targets(
          TargetGroupArn = newTargetGroupARN,
          Targets=[
              {
                  'Id': id 
              },
          ]
      )
    #print(response)
  '''  
