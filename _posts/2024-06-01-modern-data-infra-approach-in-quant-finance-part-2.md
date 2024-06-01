---
layout: post
title: Modern Data Infrastructure Approach in Quant Finance - Part 2
subtitle: What are the problems we are facing?
cover-img: 
tags: [data, dataengineering, quantfinance]
comments: true
---

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/1bad48a6-08c3-4a9b-aa88-16ec892b1739)

In the late '80s and early '90s, if you saw a company still using pens and paper instead of typewriters, you would know it was unlikely to survive for long. Over time, we've witnessed an evolution in data processing and storage: from typewriters to computers in the late '90s, from floppy disks to CD-ROMs in the 2000s, and finally from USB drives to cloud storage today. The format and medium of data storage have changed rapidly since personal computers gained popularity. The only constant has been the term "file" to describe storage units. In the following sections, we will explore how and where objects, sets of data, and mixtures of structured and unstructured data are stored in modernised data infrastructure.


## Object storage

It is interesting to reflect the history of network drive and their surprising resilience. I still recall I first saw the network drive in my first job in an actuarial team in a reinsurance company in 2010. The drive was accessible for all team members and the access control was enforced by the trust and word-of-mouth. Looking back, they were already quite advanced in employing network drives for team corporations at the time. Over the years, network drives have remained a vital component in various companies. Despite their open access making them one of the common sources of security and credential breaches, they are too convenient to abandon. 

Thanks for the prevalence of cloud platforms, object storage has become a household name. It started with Dropbox and is now widely supported by all major cloud platforms, including AWS, Google Cloud and Microsoft Azure. Although it may seem like a rebranded network drive, cloud object storage has resolved three major obstacles in managing network drives - scalability, security and performance.

One of the greatest limitations in network drives is their fixed sizes. A specific size must be allocated when creating a network drive, often resulting in either over-provisioning or under-provisioning. This leads to either paying too much for unused space or rushing to increase disk size when needed. Cloud object storage immediately solves this problem with its inherently unlimited storage capacity.

In a single network drive, all objects are stored at the same cost, charging the same price per byte regardless of importance or read frequency. Cloud object storage, however, allows policy schema that defines the storage level based on data life cycle and importance. For example, in AWS S3, a bucket can be configured to automatically archive the untouched files to the Glacier storage class, saving up to 20 times the cost. The level of flexibility was unimaginable with traditional network drives. 

|![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/97315531-59ab-44e6-99ef-06db7b4addeb)|
|:---:|
| *AWS S3 Bucket Encryption Design - Sourced from [AWS](https://aws.amazon.com/blogs/security/how-to-prevent-uploads-of-unencrypted-objects-to-amazon-s3/)* |

Access control in cloud object storage is also straightforward. In AWS S3, objects are encrypted by default, with encryption and decryption managed by Key Management System (KMS). Also, role-based access can be easily defined using simple file name patterns. Additionally, cloud object storage eliminates the need to share files via email attachments, which lack control over forwarding and sharing. Instead, shareable links with specifying access expiration can be created. Therefore, the cloud object stores provide more smarter tools to enforce modern security measures.

Last but not least, while network drives performance can be optimised for local LAN access, their performance across regions has always been a drawback. Site Reliability Engineers (SREs) often had to choose between atomicity and performance when deploying network drives. Cloud object storage, however, manages synchronisation across multiple regions. Also, cloud providers now guarantee read-after-write consistency in the object storage service, and continuous work to reduce latency in accessing object stores. 


## Evolution of Databases

Traditionally, RBDMS databases were stitched with initial values specified on computing and storage quotas. Although these limitations could sometimes be adjusted upon usages, scaling up databases usually required tremendous efforts from both data and infrastructure teams. As a result, striking a balance between pre-allocated resources and potential operational efforts, databases are often over-provisioned and underutilised.

However, advancements in cloud computing have introduced more scalable and flexible solutions. Some cloud-based RDBMS services now offer the ability to automatically scale storage resources to meet demand. These enhancements are often part of serverless infrastructure, allowing clusters to instantly scale up or down in real time based on traffic and workload. This approach offers a competitive pricing model. similar to pay-as-you-go, for both storage and computation. For example, storage is typically charged per GB of data, while computation is charged per capacity unit (such as the Accumulated Capacity Unit in AWS Aurora).

The concept of databases has now evolved into data warehouses. Data warehouses not only store large volumes of data from multiple sources, but also perform Extract, Transform and Load processes (ETL) in a managed and organised manner. To handle large volumes of data, storage is only half the solution; the other half is business intelligence. Data warehouse services are now available on nearly every cloud platform, such as Redshift in AWS Cloud and Big Query in GCP. Managed ETL platforms, provided by the cloud or solution providers, have become phenomenal for enterprises.

Simultaneously, segregating storage and query processing have become a critical feature for many data warehouse providers. Snowflake has excelled in recent years leveraging cloud computing to enhance the user experience in query processing within their data warehouse. They utilise a hybrid model of globally shared-disk but locally shared-nothing architecture. For instance, when a data provider supplies datasets in Snowflake, the data are accessible by all the nodes theoretically. With defined subscribers, the query processes stream the datasets to clusters, where each node stores a portion of data locally. It acts like magic to mirror the providers' data set to their own clusters in the customer's view. 

|![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/7d803308-8896-4d2f-b771-8e62ecd9ed46)|
|:---:|
| *Snowflake Architecture Design - Sourced from [Snowflake Documentation](https://docs.snowflake.com/en/user-guide/intro-key-concepts)* |

Furthermore, the concept of databases has evolved from data warehouses to data lakes. While data warehouses focus on storing structured data with transformation and processing power, data lakes extend to store both structured and unstructured data on a centralised platform. In the realm of artificial intelligence and machine learning, data lakes are often integrated with toolkits to facilitate ML analysis. 

|![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/f2dfd807-e4aa-4e4e-b7ed-7f2af6991d5c)|
|:---:|
|*An example of RAG on Azure Databricks - Sourced from [Microsoft Learn](https://learn.microsoft.com/en-us/azure/databricks/generative-ai/retrieval-augmented-generation)*|

However, even though data lakes are compatible with unstructured data storage, GenAI rather retrieves the data directly from data lakes to train and run the models but requires a specialised database to enhance performance. This is where vector databases come into play. A vector database stores the unstructured data objects, like text and images, in numerical vectors transformed from embedding models. During model training, vectors are conveyed from vector databases into LLMs for efficient processing. Post-training, vector databases are employed with Retrieval Augmented Generation (RAG) approach to enable storing new information for LLMs to continue learning and improve their responses. Now, vector databases are a  core component of GenAI infrastructure. 

## Bottom Line

Interestingly, despite the past decades of evolution in data storage, SQL remains the king of data query and processing languages. Business users essentially need a straightforward, resilient, and performant language to manipulate data, and SQL ticks all the boxes. Regardless of how advanced data storage has become, SQL support remains a mandatory feature for users. Sometimes, new types of databases, such as PostgreSQL as a vector database, are built on the foundation of resilient RDBMS databases. Modern managed data storage is moving away from traditional forms but is developing on top of these solid foundations.
