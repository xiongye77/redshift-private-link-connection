# AWS private link

No need for vpc peering ,no route table, no  NAT gateway/IGW

![image](https://user-images.githubusercontent.com/36766101/160306661-6f1848ab-fa8a-4cf6-a8ce-17f2500d3f2b.png)


Create AWS  VPC endpoint service for Redshift and put Redshift leader node behind NLB. 

Background information  about  What is PrivateLink

PrivateLink is a service AWS provides us. It allows a direct connection between VPCs, even if they reside in separate AWS accounts. (This is what we needed, we and vendor are in different AWS accounts)
This service does not require a VPN Peering connection. Here is a diagram illustrating the basic parts of this type of connection:
![image](https://user-images.githubusercontent.com/36766101/160306841-2383d78c-d274-4434-8337-7ea0bac5ae72.png)





Step1 : Create one NLB and register Redshift leader node to target group. Make sure NLB schema is internal and Cross-zone load balancing is Enabled. Optionally, We can enable NLB Access logs to S3 (configure bucket policy Access logs for your Network Load Balancer - Elastic Load Balancing (amazon.com)).  Register target ip of Redshift leader node and port is 5439

![image](https://user-images.githubusercontent.com/36766101/160306863-8603ac5a-9378-4d57-bd25-2753626b93c5.png)


 


Step 2: Create new VPC endpoint service for the created NLB, add “Allow principals” which will be get  from vendor.  Provide service name to vendor, let vendor connect to endpoint. 

![image](https://user-images.githubusercontent.com/36766101/160306871-998f52aa-56ff-454b-973b-e6967d04ebb4.png)

 
![image](https://user-images.githubusercontent.com/36766101/160306898-cf20198c-5b26-4548-8be5-3b4ab1d567c1.png)


After vendor make request for connection, we need accept it so vendor can connect to the endpoint 

Enabling an AWS PrivateLink between ThoughtSpot Cloud and your Redshift data warehouse | ThoughtSpot.  
![image](https://user-images.githubusercontent.com/36766101/160306934-68da8130-c095-42e8-b064-90394cef8813.png)

Step 3  Test from vendor side the connection can be established 



[admin@ip-10-229-2-142 latest]$ nc -zv vpce-0a8529c14ab7a22bc-4z6o76kx.vpce-svc-002a253210c2efb84.ap-southeast-2.vpce.amazonaws.com 5439
Ncat: Version 7.50 (  )
Ncat: Connected to 10.229.2.220:5439.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
 

Benefit of private link connection solution :

More secure since Redshift data only in AWS network and are not public available.

Bandwidth guaranteed since traffic only in AWS backbone network.  

Issues met for this solution  

After step2 is finished, we could not accomplish step3. We opened a AWS support case. following are solution provided by AWS. 

It is very important to notice that your vendor do in fact see Availability zone with name ap-southeast-2b but it is totally different then the one used by your account 555050780997 This confusion is covered on this public document ion from AWS, and you should consider it with any new VPC endpoint configuration: https://docs.aws.amazon.com/vpc/latest/privatelink/endpoint-service-overview.html#vpce-endpoint-service-availability-zones 

You can choose any of the below actions to solve the issue 

1- Move your target "Lead" in 10.240.86.29 in Availability zone ap-southeast-2b to ap-southeast-2a , and then recreate new VPC enddoint You have mentioned that this option is very hard and so you will go with option 2: 

2- Ask your vendor "the entity who has control over account 240140-489168 and vpc-0df3995c93dc23e32" to check their AZ IDs mapping Availability Zone IDs for your AWS resources - AWS Resource Access Manager And to create new subnet in the AZ that have the same "ZoneId": "apse2-az1" , then they need to create new EC2 in this subnet and you should be able to use it to access the VPC endpoint.

 
