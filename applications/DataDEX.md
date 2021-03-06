# Open Grant Proposal

* **Project Name:** DataDEX Network
* **Team Name:** DataDEX
* **Payment Address:** USDT 0x072fb7066E0ac1CfB71cAE5742Faa8879C83Fcc9



## Project Overview :page_facing_up: 

DataDEX Network is a dencentralized exchange system that tokenizes data permissions and has an automatic market-making algorithm for liquidity with AMM capabilities. 
The issuance of DataToken is created and operated by DataMaker, and is cross-chain transacted with computing tokens of other computing networks through Polkadot's parachain to maxmized value of data.

### Overview


  * Maximize the value of data tokens through the AMM mechanism, and transform data from commodities into capital owned by data owners.
  
  * Cross-chain between privacy computing parachains based on Polkadot substrate.
  
  * Leverage Task Oracle to execute tasks on other computation network, there are Preserving privacy data based on TEE or Federated Computation without moving the raw data out of data owner's devices.

### Project Details 

![图片](https://github.com/datadex-trade/Documents/blob/main/datadex_details.png?raw=true)


Features:

* Liquidation contract and Data token contract base on substrate
* Web App is the UI of exchange to swap tokens between Data token and DDT(Main token of DataDEX)
* Registration Tool and Repository are a set of services to make the private data be owned by data providers.
* Comunication Module to message together between DataDEX and Phala Network, which follows XCMP format.
* Meta Storage and DID solution

### Main modules design and implementions

#### Data Graph

Data Graph is the first data category for personal private data. Currently contains two subcategories User Profile and User Activities.
DataGraph is designed with multi-properties schema. After register the data, the Data Owner could be verified the ownership from multiple dimensions, not only the userid. If the malicious data owner just modifies a small part, other dimensions will conflict to the real data, the registration cannot be successfully submitted and cannot be successfully verified.

The following example shows that how to prevent from malicious data which is just modified small part with the real data.

```Json
{

	"register-request": [{
			"UserProfile": [{
				"Mail": [{
					"From": "abc@emm.com"
				}, {
					"Title": "Hello dude!"
				}, {
					"body-hash": "1f3870be274f6c49b3e31a0c6728957f"   //unique property defined in DataGraph
				}]
			}]
		},
		{
			"UserDid": "ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad" //unique property defined in DataGraph
		}
	]
}
```
The malicious owner modified the UserDid to his own id, and repost the request with modified info.
```Json
{

	"register-request": [{
			"UserProfile": [{
				"Mail": [{
					"From": "abc@emm.com"
				}, {
					"Title": "Hello dude!"
				}, {
					"body-hash": "1f3870be274f6c49b3e31a0c6728957f"  //would conflicts with the real data due to the uniqueness
				}]
			}]
		},
		{
			"UserDid": "730f75dafd73e047b86acb2dbd74e75dcb93272fa084a9082848f2341aa1abb6"
		}
	]
}
```
The register request will be rejected due to the confliction.

#### Data Graph DAO

DataGraphDAO designed to be a on-chain mechanisms that could encourage community members to participate in the design of the Data Catalog and maintain the legality and availibility of data and registration.

We will reward community members who voluntarily find malicious data or errors (we hope that Dataset Maker will play a leading role), based on data characteristics analysis and relevant in data technology. and then owners with data faults should be punished. We have rich experience in anti fake products for e-commerce platforms with this similar technology.

If the data owner could not guarantee the data availability, including the data non-exists or the data unregistered etc. The tasks scheduled by the computing node(Phala, Alita network etc.) to the device of data owner will fail, DataDEX can not check the verification of task, so the DataOwner can not get the rewards.

#### Data Registeration Entry 

 * RegisterData
```Java
    /**
     * Register the copyright of the data to DataGraph
     * 
     * @param userDid       The user ID
     * @param dataCatalog   The path in data graph catalog
     * @param dataBody      The jsonized structure of data description
     */
    public void registerData(UserDID userDid, DataGraph dataCatalog, String dataBody);
```
 * CheckPermissions

```Java
    /**
     *  Check accessibility of data with userid
     * 
     * @param userDid       The user ID
     * @param dataCatalog   The path in data graph which will access
     * @return boolean
     * 
     */
    public boolean registerData(UserDID userDid, DataGraph dataCatalog);
```
 * DataGraph ProtoBuf example
```Java
    syntax = "proto3";
    
    option java_package = "com.datadex.datacatalog";
    option java_outer_classname = "DataGraph";
    
    message UserProfile {
        String userdid = 1;
        Mail mail = 2;
        Notes notes = 3;
        Notifications notifications = 4;
        PersonContacts contacts = 5;
        TasksAndPlans schedule = 6;
        Messages sms = 7;
        Applications apps = 8;
        ...
    }
    
    message Applications{
        ...
    }
    ...
```
#### Task Oracle

![图片](https://github.com/datadex-trade/Documents/blob/main/taskoracle.png?raw=true)

* **TaskMaster** is the task queue as pallets on chain which could be leveraged for Task scheduling. 
* **TaskExecutor** is an off-chain node for task execution, which would be deployed through locality within data graph chunk data on each user device.
* **DataCheckerNode** is a data monitoring node that guarantees permission checking and data availability.
* **Graph Raw Data** is the chunk data identified by each user, to make user data only could be stored in user edge device.

Based on the architecture, computing tasks could process data on user devices, and Checking Task could also check data graph for legality of registration request.

#### Simple orchestration flow

![图片](https://github.com/datadex-trade/Documents/blob/main/DataDEXflow.png?raw=true)

#### Scenario example: Data Owners share personal medical data to get benefits
···
0. Preparation
Data Owner download medical record data to local device through [FHIR][FHIR]-based applications on [Android Phone][FHIR_android] , [iPhone][FHIR_ios] or [Windows PC][FHIR_azure].

1. Register Medical Data
Use the Registration Tool to transform the data into the UserProfile.MedicalRecord format in the DataGraph and submit the request.
The RegistrationTool submits the request to TaskOracle to trigger the Legality Checking Task, and is dispatched to the Data Owner's device to check with the raw data.

   1.1 malicious checking, if the Data Owner steals other people’s data and only modifies a small part of the submission, the Legality Checking Task will find that it is very similar to the existing data, Checking Fail

   1.2 If the Legality Checking Task passes the check, the registration request and data hash are broadcast, and the DataGraph is updated.

2. Dataset Maker, who are experiences at medical data operations, invite Data Owner to contribute data to the Dataset and get DataToken as rewards.

3. Pharmaceutical R&D companies purchase DataToken to run training tasks, and schedule tasks to user devices through TaskOracle, and access raw data in DataGraph.

   3.1 If the data is not available, the Task should failed to run
   
   3.2 If the data is not correct, the hash checking would not be verified. The task should failed.

   3.2 If the data is available, the Task runs successfully, and the DataOwner gets the DataToken incentive
···
### Data Pricing

#### Data Token AMM

DataDEX provides automated price discovery with AMM, which works for an initial dataset offering and throughout the dataset’s lifetime. AMMs can automatically match buyers and sellers based on automatic price discovery and can improve market-making efficiency and transaction volume.

Only large-scale data collections are valuable, and individual data fragments have limited value. DatasetMaker will create a contract on the main chain and list the token of this data (Data Token) collection with  pairs to a new pool(Dataset). In order to maximized obtain the value of data, DataOwner need to join a certain Dataset and submit Metadata to get the rewards sent by DatasetMaker. The number of liquidity tokens minted is computed based on the existing quantity of tokens. A dataset maker no longer specifies which prices they are willing to sell. Instead, dataset pools everyone’s data supply and demand liquidity together and makes markets according to a deterministic algorithm.

![图片](https://github.com/datadex-trade/Documents/blob/main/datadex_pricing.png?raw=true)

#### Data Token Costs of Computing

Data Consumer first needs to prepare a certain amount of PHALA/ALITA to pay for computing costs. Then use DDT to buy the Data Token. Based on the privacy-preserving computation like PHALA Network or Alita Network, etc. Data-owner’s plain-text data is not collected outside the device, but execute outsourced tasks on the device provided by Data-owner. Therefore, the value of data can be measured by complexity of the computation tasks. Caculate how much PHALA/ALITA the task consumes, and consume the equivalent Data Token.

Data Token Costs = K * (PHA/ALITA/...)

Total Costs = Data Token Costs + Computing Costs

In addition, cost calculation and data price measurement are already common pricing strategies in cloud computing like AWS [data infrastructure][Spectrum_pricing] and [data exchange service][Exchange_Subscription].

### Ecosystem Fit 

We would like to cooperate with the following ecological projects and platforms to enhance the development progress:

* [Moonbeam](https://moonbeam.network/) is a new Polkadot smart contract platform that makes it easy to build natively interoperable blockchain applications. 
* [Phala Network](https://phala.network/) is a privacy-preserving cloud computing service, which offers computing power comparable to existing cloud services and protects the privacy of managed programs.

## Team :busts_in_silhouette:

### Team members

* Hongyu Luo:Former Senior Cloud Expert at Alibaba and Technical Director of LeTV. Seven years of experience at Alibaba Cloud as responsible for the Alibaba Cloud data computing platform, product development and commercialization of the 11.11 big data parallel processing platform, supporting thousands of brands, service providers and public cloud enterprises.
* Jingsong Yi: Bachelor of Information Management and Information System at Henan Polytechnic University. Extensive experience in design and development of world leading decentralized joint edge computing systems based on mobile devices, data analysis and computation in Trusted Execution Environment (TEE).
* Hui Bao: Graduated from Harbin University Of Commerce. Master in Solidity and blockchain technology. Extensive experience with J2EE and Distributed Computing systems.


### Contact
* **Contact Name:** Hongyu Luo
* **Contact Email:** luohongyu@datadex.trade
* **Website:**  https://datadex.trade

### Legal Structure 
* **Registered Address:** 60 PAYA LEBAR ROAD #08-55 PAYA LEBAR SQUARE SINGAPORE
* **Registered Legal Entity:** Data Chain Foundation Ltd.

### Team's experience

Alita.global implemented a MapReduce framework supporting Android system can handle complex edge end data analysis task rapidly, so that provides customers with low cost and privacy-preserving data computing service.[To download](http://app.gravity.top:8081/apk/alita_enc_sign.apk).

Alita network accounting system is based on Proof of Capacity blockchain, on which we built a computing marketplace with smart-contract.[Visit Testnet](http://testnet.alita.services:8304/index/)

There is also a web IDE for data development and computing resources management.

### Team Code Repos

This is an open source project under Apache License 2.0. All the defined milestones will be available to the open source community.


### Team LinkedIn Profiles



## Development Status :open_book: 

At present, we have developed some prototype systems based on other public chains, and plan to migrate all of them to the substrate-based blockchain.


## Development Roadmap :nut_and_bolt: 

### 

### Overview

### Milestone 1

* Total Estimated Duration: 1.5 months
* Full-time equivalent (FTE): 3
* Total Costs: 13000 USD

|Number|Deliverable|Specification|
|:----|:----|:----|
|0a.|License|Apache 2.0 / MIT / Unlicense|
|0b.|Documentation|We will provide a basic tutorial that explains how a user can share privacy data safely. Once the node is up, it will be possible to send test transactions that will show how the new functionality works.|
|0c.|Testing Guide|In the guide we will describe how to run these tests|
|0d.|Article/Tutorial|We will write an article or tutorial that explains the work done as part of the grant.|
|1|Graph Schema|Design user graph data schema, including activities and profile. Defined in Proto3, and will store to an object storage like ipfs. In future, there will be a graph engine to analyse the data as iterative computation|
|2|Data Reisteration Entry|Deliver a mobile app by which users could register personal data to data graph. Develop in Kotlin, Android and ios version. And provide rest api devlop in NodeJs.|
|3|Frontend|Data DEX web system development. Vue based H5 pages involved in mobile app.|

### Milestone 2
* Total Estimated Duration: 1.5 months
* Full-time equivalent (FTE): 3
* Total Costs: 12000 USD

|Number|Deliverable|Specification|
|:----|:----|:----|
|1|Data Graph DAO|Develop DAO contract in ink! and managenment tools develop in Kotlin, Android and ios version. |
|2|Smart Contract|Data DEX smart contracts development with ink!|
|3|Task Oralce|Oracle development for receiving data computing tasks, adapted to the computing tasks of Phala and Alita Network. Develop in Rust as a runtime pallets.|



## Future Plans

* Continuously improve Graph Schema to make it a complete personal data confirmation catalog in the Polkadot eco-system
* Promote personal data investment in data set construction and enrich the data content of the data market
* Gradually introduce business data consumers, and continuously improve the volume and value of data in the market
* Cooperating with more computing chains, to make the data is safely shared within different privacy-preserving computation protocols
### 


## Additional Information :heavy_plus_sign: 

## References

[Spectrum_pricing]: https://aws.amazon.com/redshift/pricing/#Redshift_Spectrum_pricing  "AWS Redshift Spectrum pricing"
[Exchange_Subscription]: https://aws.amazon.com/data-exchange/faqs/#Subscriber "AWS Data Exchange Subscription"
[FL_Dataset]: https://ai.googleblog.com/2017/04/federated-learning-collaborative.html "Google FL: One High Quanlity Dataset Distributed across millions of Phones"
[FHIR_android]: https://cloud.google.com/healthcare/docs/concepts/fhir "Goole Cloud API for FHIR"
[FHIR_ios]: https://www.apple.com/healthcare/health-records/  "Apple Health Built with industry standards FHIR"
[FHIR_azure]: https://azure.microsoft.com/en-us/services/azure-api-for-fhir/ "Azure API for FHIR"
[FHIR]: https://en.wikipedia.org/wiki/Fast_Healthcare_Interoperability_Resources "Fast Healthcare Interoperability Resources"
