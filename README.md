<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Azure Peering

### [&rarr; Azure Peering](#Peering)
### [&rarr; Grant Permission to Peering Connection](#Permission)

<br>

### Peering
Atlas Cluster와 Azure networ간 Public network를 이용하지 않고 직접 연결(Peering)을 합니다.

Atlas Console에 로그인 후에 Network Access 메뉴를 클릭하고 Peering 에서 Add Peering을 클릭 합니다.   

<img src="/images/image01.png" width="60%" height="60%">     

Azure를 선택 한 후 연결 대상 네트워크 정보를 입력 하여야 합니다. (Azure Console에서 데이터를 얻을 수 있습니다)   

<img src="/images/image02.png" width="90%" height="90%">     

Azure Console에 로그인 후 Subscription을 입력 하여 줍니다.    

<img src="/images/image03.png" width="90%" height="90%">     

Active Directory Tenant ID를 입력 하여 줍니다.    

<img src="/images/image04.png" width="90%" height="90%">     

자원(Virtual Network)이 소속된 Resource Group 이름을 입력 하여 줍니다.    

<img src="/images/image05.png" width="90%" height="90%">     

연결 대상 Network 이름을 입력 하여 줍니다.    

<img src="/images/image06.png" width="90%" height="90%">     

입력을 완료 한 후 대상 네트워크의 리전을 선택 하고 다음을 선택 합니다.    

<img src="/images/image07.png" width="90%" height="90%">     


### Permission

네트워크 연결을 위해 권한을 설정 해야 합니다. 이를 수행 하기 위한 Permission Code 작성 스크립트가 보여지며 이를 실행 하여 줍니다.     

<img src="/images/image08.png" width="90%" height="90%">     


Azure CLI 가 실행 가능한 환경에서 코드를 차례로 수행 하여 줍니다.

````
kim [ ~ ]$ az ad sp create --id e90a1407-55c3-432d-9cb1-3638900a9d22
kim [ ~ ]$ vi peering-role.json 
kim [ ~ ]$ az role definition create --role-definition peering-role.json
Readonly attribute type will be ignored in class <class 'azure.mgmt.authorization.v2022_04_01.models._models_py3.RoleDefinition'>
{
  "assignableScopes": [
    "/subscriptions/15b444cb-e1ff-4319-****-**********/resourceGroups/mdb/providers/Microsoft.Network/virtualNetworks/mongo-vnet"
  ],
  "description": "Grants MongoDB access to manage peering connections on network /subscriptions/15b444cb-e1ff-4319-****-**********/resourceGroups/mdb/providers/Microsoft.Network/virtualNetworks/mongo-vnet",
  "id": "/subscriptions/15b444cb-e1ff-4319-****-**********/providers/Microsoft.Authorization/roleDefinitions/efc08076-6b02-476d-aa1d-2309693e6787",
  "name": "efc08076-6b02-476d-aa1d-2309693e6787",
  "permissions": [
    {
      "actions": [
        "Microsoft.Network/virtualNetworks/virtualNetworkPeerings/read",
        "Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write",
        "Microsoft.Network/virtualNetworks/virtualNetworkPeerings/delete",
        "Microsoft.Network/virtualNetworks/peer/action"
      ],
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "AtlasPeering/15b444cb-e1ff-4319-****-**********/mdb/mongo-vnet",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
kim [ ~ ]$ az role assignment create --role "AtlasPeering/15b444cb-e1ff-4319-****-**********/mdb/mongo-vnet" --assignee "e90a1407-55c3-432d-9cb1-3638900a9d22" --scope "/subscriptions/15b444cb-e1ff-4319-****-**********/resourceGroups/mdb/providers/Microsoft.Network/virtualNetworks/mongo-vnet"
{
  "condition": null,
  "conditionVersion": null,
  "createdBy": null,
  "createdOn": "2023-06-07T15:58:03.107939+00:00",
  "delegatedManagedIdentityResourceId": null,
  "description": null,
  "id": "/subscriptions/15b444cb-e1ff-4319-****-**********/resourceGroups/mdb/providers/Microsoft.Network/virtualNetworks/mongo-vnet/providers/Microsoft.Authorization/roleAssignments/8136f375-5f3a-499b-9d58-c1a33744dbaf",
  "name": "8136f375-5f3a-499b-9d58-c1a33744dbaf",
  "principalId": "78257de6-644f-4284-99dd-c02497b8bb7a",
  "principalType": "ServicePrincipal",
  "resourceGroup": "mdb",
  "roleDefinitionId": "/subscriptions/15b444cb-e1ff-4319-****-**********/providers/Microsoft.Authorization/roleDefinitions/efc08076-6b02-476d-aa1d-2309693e6787",
  "scope": "/subscriptions/15b444cb-e1ff-4319-****-**********/resourceGroups/mdb/providers/Microsoft.Network/virtualNetworks/mongo-vnet",
  "type": "Microsoft.Authorization/roleAssignments",
  "updatedBy": "22fdfb2d-245f-4c60-94f6-79c99a6763d8",
  "updatedOn": "2023-06-07T15:58:04.481484+00:00"
}
````

실행 완료 후 Atlas console에서 validation을 실행 하여 줍니다.   

<img src="/images/image09.png" width="90%" height="90%">     

저장이 완료되면 Peering 상태가 available 로 변경 됩니다.    

<img src="/images/image10.png" width="90%" height="90%">    

Virtual network 에 VM을 생성 하고 연결 테스트를 진행 합니다.    
Cluster에서 Connection 버튼을 클릭 합니다.    

연결 정보에서 Private IP for Peering을 선택 합니다.   

<img src="/images/image11.png" width="80%" height="80%">     

Mongosh 로 테스트 할 것임으로 Shell을 선택 합니다.   

<img src="/images/image12.png" width="70%" height="70%">     

접속 주소를 복사 한 후 VM terminal에서 실행 하여 줍니다.

````
[azureuser@mongo ~]$ mongosh "mongodb+srv://azuretest-pri.ABCDE.mongodb.net" --apiVersion 1 --username myAtlasUser
Enter password: **********
Current Mongosh Log ID: 6480ac439de0b8087acb9822
Connecting to:          mongodb+srv://<credentials>@azuretest-pri.ABCDE.mongodb.net/?appName=mongosh+1.9.1
Using MongoDB:          6.0.6 (API Version 1)
Using Mongosh:          1.9.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

Atlas atlas-13urby-shard-0 [primary] test>

````

실제 접속 주소가 Private IP로 되는지 확인 하기 위해 다음을 수행 합니다.     

````
[azureuser@mongo ~]$ dig srv _mongodb._tcp.azuretest-pri.ABCDE.mongodb.net
; <<>> DiG 9.11.36-RedHat-9.11.36-3.el8 <<>> srv _mongodb._tcp.azuretest-pri.ABCDE.mongodb.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 162
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1224
;; QUESTION SECTION:
;_mongodb._tcp.azuretest-pri.ABCDE.mongodb.net. IN SRV

;; ANSWER SECTION:
_mongodb._tcp.azuretest-pri.ABCDE.mongodb.net. 60 IN SRV 0 0 27017 azuretest-shard-00-00-pri.ABCDE.mongodb.net.
_mongodb._tcp.azuretest-pri.ABCDE.mongodb.net. 60 IN SRV 0 0 27017 azuretest-shard-00-01-pri.ABCDE.mongodb.net.
_mongodb._tcp.azuretest-pri.ABCDE.mongodb.net. 60 IN SRV 0 0 27017 azuretest-shard-00-02-pri.ABCDE.mongodb.net.

;; ADDITIONAL SECTION:
azuretest-shard-00-00-pri.ABCDE.mongodb.net. 60 IN A 192.168.254.5
azuretest-shard-00-01-pri.ABCDE.mongodb.net. 60 IN A 192.168.254.4
azuretest-shard-00-02-pri.ABCDE.mongodb.net. 60 IN A 192.168.254.6

;; Query time: 4 msec
;; SERVER: 168.63.129.16#53(168.63.129.16)
;; WHEN: Wed Jun 07 16:12:53 UTC 2023
;; MSG SIZE  rcvd: 311
````

접속 주소가 192.168.254.4 ~ 192.168.254.6 으로 Public 주소가 아닌 내부 주소로 접속 하는 것을 확인 할 수 있습니다.    