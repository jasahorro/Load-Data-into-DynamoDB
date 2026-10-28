<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Load Data into DynamoDB

**Project Link:** [View Project](https://nextwork.ai/projects/8bd82466-f5ee-569e-9125-2c4c6ce7a07f)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_b481c730)

## Introducing Today's Project!

### What is Amazon DynamoDB?

Amazon DynamoDB is a fully managed, serverless NoSQL database service provided by AWS that stores data as flexible items and attributes rather than using traditional rigid rows and columns. It is exceptionally useful because its schemaless architecture allows developers to dynamically evolve data structures without running complex table migrations or enforcing uniform schemas. Furthermore, DynamoDB uses primary partition keys to directly index and retrieve data, delivering lightning-fast, single-digit millisecond performance even as data scales to millions of requests per second. Because AWS handles underlying hardware provisioning, patching, and availability, engineering teams can focus entirely on building high-performance applications. Combined with a generous free tier covering 25GB of storage and automated scaling capabilities, DynamoDB provides a scalable, cost-effective, and efficient database solution for modern, high-traffic web and mobile applications.

### How I used Amazon DynamoDB in this project

In today's project, I used Amazon DynamoDB to provision and manage non-relational database tables, exploring how NoSQL architectures store data as flexible items and attributes rather than rigid rows and columns. I started by creating a student database table manually through the AWS Management Console with customized read and write capacity units to stay within the AWS Free Tier. Afterward, I utilized AWS CloudShell and the AWS CLI to run batch write operations that efficiently bulk-loaded sample dataset files containing educational content, forums, posts, and comments into multiple DynamoDB tables. Finally, I inspected and edited individual item attributes, gaining hands-on experience in how DynamoDB delivers high-speed performance and exceptional schema flexibility compared to traditional relational databases.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project is how seamlessly Amazon DynamoDB handles data flexibility compared to traditional relational databases. Coming from a background of working with rigid schemas, I was genuinely surprised to see that items within the exact same table can maintain completely unique and independent attributes without needing predefined columns or complex table migrations. Furthermore, being able to launch AWS CloudShell directly from the console with the AWS CLI pre-configured made bulk-loading entire JSON datasets feel remarkably fast and efficient. This hands-on experience completely changed how I think about managing and structuring non-relational data at scale.

### This project took me...

This project took me 30 minutes to complete.

## Create a DynamoDB table

DynamoDB tables organise data using items and attributes rather than traditional rows and columns, allowing each individual item to maintain its own unique and flexible set of attributes. Instead of using a strict relational schema where every row shares the exact same predefined columns and headers, DynamoDB relies on a primary partition key to efficiently organize and locate data. This flexible NoSQL structure means that different items within the same table can easily hold completely different fields, making it ideal for scalable, fast-moving applications that handle diverse data types without sacrificing performance or needing complex table migrations.

An attribute is a single piece of data or a specific field that describes an item in a DynamoDB table (such as the number of projects completed or a student's name).

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_a3cefee0)

## Read and Write Capacity

Read capacity units (RCUs) and write capacity units (WCUs) are performance measurements used by Amazon DynamoDB to determine how many read and write operations your database can handle per second (where 1 RCU allows up to 2 reads per second for items up to 4KB, and 1 WCU allows 1 write per second for items up to 1KB).

Amazon DynamoDB's Free Tier covers up to 25GB of data storage, plus 25 Write Capacity Units and 25 Read Capacity Units per month, I turned off auto scaling because it can automatically boost your table's processing power and push your settings beyond the Free Tier limits if not closely monitored.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_ef47dd8f)

## Using CLI and CloudShell

AWS CloudShell is a browser-based, pre-authenticated shell environment built directly into the AWS Management Console that allows developers and cloud engineers to run scripts and execute AWS CLI commands without needing to install or configure tools locally. Because it comes with the AWS Command Line Interface (CLI) pre-installed and pre-configured with your console credentials, you can immediately begin managing and provisioning cloud resources, such as creating Amazon DynamoDB tables or downloading dataset files, straight from your browser.

AWS CLI is an open-source command-line tool provided by Amazon Web Services that allows developers and cloud engineers to interact with, manage, and provision AWS resources—such as creating and updating DynamoDB tables—directly using text commands in a terminal rather than clicking through the graphical AWS Management Console.

I ran a CLI command in AWS CloudShell that created four distinct Amazon DynamoDB tables (ContentCatalog, Forum, Post, and Comment) using the AWS CLI create-table command. Each table was configured with specific attribute definitions and partition or sort keys—such as a numeric Id for the content catalog, a Name partition key for forums, and combination primary keys for posts and comments—alongside provisioned read and write capacities set to 1 to stay safely within the AWS Free Tier.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_81e0258b)

## Loading Data with CLI

I ran a CLI command in AWS CloudShell that used the aws dynamodb batch-write-item command to bulk-load all our sample dataset records from the downloaded JSON files (ContentCatalog.json, Forum.json, Post.json, and Comment.json) directly into their respective DynamoDB tables, allowing us to populate the database efficiently without entering items manually.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_791c600b)

## Observing Item Attributes

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_b481c731)

I checked a ContentCatalog item, which had the following attributes: an Id set to 1 serving as the partition key, a Title attribute indicating the name of the project, a ContentType attribute set to "Project", a Difficulty level, and additional metadata attributes like category and description.

I checked another ContentCatalog item, which had a different set of attributes... specifically a video item with an Id set to 203, containing attributes such as Title, URL, and VideoType set to "Live Project Demo", highlighting how different items in a DynamoDB table can maintain their own unique fields instead of sharing a rigid schema.

## Benefits of DynamoDB

A benefit of DynamoDB over relational databases is flexibility, because its schemaless NoSQL architecture allows every individual item in a table to maintain its own unique and distinct set of attributes. Unlike traditional relational databases—which enforce a rigid, uniform schema where every single row must share the exact same predefined columns and headers, often resulting in wasted storage space and mandatory table migrations when data structures change—DynamoDB enables applications to store diverse, rapidly evolving data types side-by-side. This means you can seamlessly add new attributes to specific items without affecting others, making it the ideal choice for modern, fast-paced applications handling varied and dynamic datasets.

Another benefit over relational databases is speed, because Amazon DynamoDB utilizes primary partition keys to quickly index, locate, and retrieve data directly rather than forcing the system to perform time-consuming full table scans. In traditional relational databases, queries often require complex joins across multiple rigid tables and rows, which can dramatically slow down performance as data volume scales. DynamoDB's streamlined NoSQL key-value architecture cuts through this overhead by mapping requests straight to specific partitions. Furthermore, because items maintain flexible attributes without needing uniform schemas or heavy relational mapping, read and write operations execute with minimal latency. This high-speed performance makes DynamoDB exceptionally well-suited for fast-moving, high-traffic applications that require instant data retrieval and seamless scalability under heavy workloads.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8bd82466-f5ee-569e-9125-2c4c6ce7a07f_b481c730)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/8bd82466-f5ee-569e-9125-2c4c6ce7a07f)*
